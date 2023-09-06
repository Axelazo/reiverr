<script lang="ts">
	import { settings } from '$lib/stores/settings.store';
	import { addMessages, init, locale } from 'svelte-i18n';

	interface LocaleDictionary {
		[key: string]: LocaleDictionary | string | Array<string | LocaleDictionary> | null;
	}

	settings.subscribe((value) => {
		if (value.language) {
			locale.set(value.language);
		} else {
			locale.set('en');
		}
	});

	const importMessages = async () => {
		const messageFiles = import.meta.glob('../../lang/*.json');

		for (const path in messageFiles) {
			if (messageFiles.hasOwnProperty(path)) {
				const fileName = path.match(/\/([^/]+)\.json$/)?.[1] || '';
				const messages = await messageFiles[path]();

				// Check if the filename contains a hyphen to detect language-country variations
				if (fileName.includes('-')) {
					// Use the full filename as the key (e.g., pt-br.json)
					const language = fileName.split('.')[0];
					addMessages(language, messages as LocaleDictionary);
				} else {
					// Use the language part only (e.g., es.json, en.json, it.json, fr.json)
					const language = fileName.split('.')[0];
					addMessages(language, messages as LocaleDictionary);
				}
			}
		}
	};

	importMessages();

	init({
		initialLocale: $settings.language,
		fallbackLocale: 'en'
	});
</script>
