<!--
	MDBox.vue
	---------

	Renders markdown w/ our lib
-->
<template>
	<div class="markdown-box">
		<section class="markdown-body" v-html="renderedMarkdown"></section>
	</div>
</template>

<script setup>

// vue
import { ref, onMounted, useSlots } from 'vue'

// markdown library
import MarkdownIt from 'markdown-it'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css'
import '@/assets/markdown.css'

// the actual markdown file to render
const slots = useSlots();

// the code rendered
const renderedMarkdown = ref('')


// set up mark down renderer when we mount the component
onMounted(async () => {

	// emoji errors, ignored for now
	const emoji = await import('markdown-it-emoji').then(m => m.default || m)
	const container = await import('markdown-it-container').then(m => m.default || m)
	const anchor = await import('markdown-it-anchor').then(m => m.default || m)

	const md = new MarkdownIt({
		html: true,
		linkify: true,
		typographer: true,
		highlight: (str, lang) => {
			let clean = str.replace(/^\s*\n/, '').replace(/\s+$/, '').trimStart()
			if (lang && hljs.getLanguage(lang)) {
				try {
					return `<pre class="hljs"><code>${hljs.highlight(clean, { language: lang }).value}</code></pre>`
				} catch (_) {}
			}
			return `<pre class="hljs"><code>${md.utils.escapeHtml(clean)}</code></pre>`
		}
	})
	.use(container, 'tip')
	.use(anchor);

	const rawMd = extractRawTextFromSlot();
	renderedMarkdown.value = md.render(rawMd);
});


/**
 * Extracts raw text from the default slot content.
 */
function extractRawTextFromSlot() {

	const slotContent = slots.default?.() || [];

	// Concatenate all text content from the VNodes
	return slotContent.map(vnode => {
		if (typeof vnode.children === 'string')
			return vnode.children;
		
		return '';
	}).join('').trim();
}

</script>
<style lang="scss" scoped>

	.markdown-box {

		background: white;

		max-width: 1200px;
		margin: 0 auto;
		padding: 20px;
	}

</style>
