<script>
    import { marked } from 'marked';
    import { replaceMarkdown, replaceNewLine } from '$lib/helpers/http';

    /** @type {string} */
	export let text;

	/** @type {string} */
	export let containerClasses = "";

	/** @type {string} */
	export let containerStyles = "";

	/** @type {boolean} */
	export let rawText = false;

    let displayText = '';
	$: {
		const markedText = !rawText ? replaceNewLine(marked(replaceMarkdown(text || ''))?.toString()) : marked(text || '')?.toString();
		if (!!markedText && markedText.endsWith('<br>')) {
			const idx = markedText.lastIndexOf('<br>');
			displayText = markedText.substring(0, idx);
		} else {
            displayText = markedText;
        }
	}
</script>

<div class={`markdown-container markdown-lite ${containerClasses || 'text-white'}`} style={`${containerStyles}`}>
	{@html displayText}
</div>