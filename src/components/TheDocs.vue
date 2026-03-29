<!--
	TheDocs.vue
	-----------

	Renders the document .md file.
-->
<template>
	<div class="markdown-box">
		<section class="markdown-body" v-html="renderedMarkdown"></section>
	</div>
</template>

<script setup>

// vue
import { ref, onMounted } from 'vue'

// markdown library
import MarkdownIt from 'markdown-it'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css'
import '@/assets/markdown.css'

// the actual markdown file to render
import rawMd from '../docs.md?raw'

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

	renderedMarkdown.value = md.render(rawMd);
});

</script>
<style lang="scss" scoped>

	.markdown-box {

		background: white;

		max-width: 1200px;
		margin: 0 auto;
		padding: 20px;

		overflow-x: clip;
	}

</style>
