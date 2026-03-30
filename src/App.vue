<!--
	App.vue
	-------

	Main base component for the site/app.
-->
<template>

	<!-- box that renders 3D objects & interactively lets you test queries -->
	<a id="demo"/>
	<br><br><br><br>
	<IntroVideoBox
		video-url="Demo-Reel.mp4"
		top-blurb="Check out the demo reel below & scroll down to learn how to use vue-settings-panel!"
	/>

	<!-- overview video -->
	<a id="overview"/>
	<SectionHeader>Overview</SectionHeader>
	<br><br>
	<div class="overview-box">

		<div align="center">
			<img class="logo" src="/img/VSP_logo.png" alt="Vue Win Mgr Logo" width="200px"/>
		</div>

		<h2>vue-settings-panel In a Nutshell:</h2>
		<br>
		<p>vue-settings-panel is really easy to set up and use in your Vue3 projects.</p>
		<p>First, after installing the library you'll want to import the important bits:</p>
		<br>
		<img src="/img/overview_00_imports.png" alt="Code Snippet Showing Imports" width="100%"/>
		<br>
		<p>Next you need to define a specification object, with categories for settings defined in an array, as well as an object of settings definitions.</p>
		<br>
		<img src="/img/overview_10_define_spec.png" alt="Define Specification" width="100%"/>
		<br>
		<p>You can also use the optional <code>createSttings</code> method to make an object with all your settings and default values prefilled.</p>
		<br>
		<p>If you wish to listen to changes to settings, you can also pass an <code>@settings-changed</code> callback to the <code>VueSettingsPanel</code> component.</p>
		<img src="/img/overview_20_optional.png" alt="Optional Settings" width="100%"/>
		<br>
		<p>Lastly, just drop the <code>&lt;VueSettingsPanel /&gt;</code> component into your template.</p>
		<p>You can also provide a <code>:theme-colors="{}"</code> prop to customize the colors of the panel if you want.</p>
		<br>
		<img src="/img/overview_30_template.png" alt="Template Example" width="100%"/>
		<br>

		<br>
		<h2>vue-settings-panel Tutorial:</h2>
		<br>
		<p>
			Below is a full tutorial for how to get started with with the <code>vue-settings-panel</code>!
		</p>
		<br>
		<div align="center" style="min-height: 60vh;">
			<iframe
				width="100%"
				style="width: 100%; aspect-ratio: 16/9; max-width: 1200px;"
				src="https://www.youtube.com/embed/lzsGuybSp0Y?si=_8oncldvIKGlMmLv"
				title="YouTube video player"
				frameborder="0"
				allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin"
				allowfullscreen
			></iframe>
		</div>
	</div>
	
	<!-- markdown parsed & rendered from the readme -->
	<a id="get"/>
	<SectionHeader>Get</SectionHeader>
	<br><br>
	<div class="get-box">

		<h2>Install in your Project:</h2>
	
		<br><br>
		<h3>With NPM:</h3>	
		<pre class="code">npm install vue-settings-panel</pre>
		Then...
		<pre class="code">
import { VueSettingsPanel, TYPES, createSettings } from 'vue-settings-panel';
import 'vue-settings-panel/dist/style.css';</pre>

		<br><br>
		<h2>Check out vue-settings-panel over here:</h2>
		<div class="row">

			<div class="box">
				<div>
					Find it on
					<a href="https://www.npmjs.com/package/vue-settings-panel" target="_blank">NPM:</a>
				</div>
				<a href="https://www.npmjs.com/package/vue-settings-panel" target="_blank">
					<img class="site-logo" src="/img/npm_logo.png" alt="NPM Logo" height="70px"/>
				</a>
			</div>

			<div class="box">
				<div>
					Find it on
					<a href="https://github.com/orokro/vue-settings-panel" target="_blank">GitHub:</a>
				</div>
				<a href="https://github.com/orokro/vue-settings-panel" target="_blank">
					<img class="site-logo" src="/img/github_logo.png" alt="GitHub Logo" height="100px"/>
				</a>
			</div>

		</div>

	</div>

	<!-- markdown parsed & rendered from the readme -->
	<a id="docs"/>
	<SectionHeader>Docs</SectionHeader>
	<br><br>
	<TheDocs />

	<!-- bar long the top w/ links and whatnot -->
	<Header/>

	<br><br>
	<br><br>
	<div class="copy-right">&copy; Greg Miller / Orokro, 2025</div>
	<br><br>
</template>
<script setup>

// vue
import { ref } from 'vue';

// components
import Header from './components/Header.vue';
import TheDocs from './components/TheDocs.vue';
import SectionHeader from './components/SectionHeader.vue';
import MDBox from './components/MDBox.vue';
import IntroVideoBox from './components/IntroVideoBox.vue';

</script>

<style lang="scss">

	header {
		line-height: 1.5;
	}

	.logo {
		display: block;
		margin: 0 auto 2rem;
	}

	@media (min-width: 1024px) {
		header {
			display: flex;
			place-items: center;
			padding-right: calc(var(--section-gap) / 2);
		}

		.logo {
			margin: 0 2rem 0 0;
		}

		header .wrapper {
			display: flex;
			place-items: flex-start;
			flex-wrap: wrap;
		}
	}

	// add gradient & titled images to main page background
	body {

		/* background: linear-gradient(#5e5e5e, #000);
		background-position: fixed; */

		overflow-x: clip;

		// use pseudo element to add a vertical gradient that doesn't scroll with content
		&::before {

			// required content to make pseudo element work
			content: "";

			// fill screen
			position: fixed;
			inset: 0px 0px 0px 0px;
			width: 100vw;
			height: 100vh;

			// just a nice gradient
			background: linear-gradient(#f3fdff, #bcced1);

			// always on bottom, no interaction
			z-index: -1;
			pointer-events: none;	
		}
	
		// use another pseudo element to add a tiled background image
		// that doesn't scroll with content
		&::after {

			// required content to make pseudo element work
			content: "";
			opacity: 0.33;
			// fill screen
			position: fixed;
			inset: 0px 0px 0px 0px;
			/* width: 100vw;*/
			height: 100vh; 

			// tiled background image over the screen
			background-image: url('/img/bg_tiles.png');
			background-size: 602px;
			background-repeat: repeat;
			background-position: center;

			// always on bottom, no interaction
			z-index: -1;
			pointer-events: none;	

		}// &::after

	}// body

	.get-box {

		background: white;

		max-width: 1200px;
		margin: 0 auto;
		padding: 20px;

		overflow-x: clip;
		

		.row {
			display: flex;
			justify-content: space-between;
			flex-wrap: wrap;
			gap: 20px;

			.box {
				flex: 1 1 calc(50% - 20px);
				background: #f0f0f0;
				border-radius: 10px;
				padding: 20px;
				text-align: center;

				// Center content inside the box
				display: flex;
				flex-direction: column;
				align-items: center;
				justify-content: center;
				text-align: center;

				.site-logo {
					margin-top: 20px;
				}

			}// .box

		}// .row

	}// .get-box

	// the overview section
	.overview-box {

		background: white;

		max-width: 1200px;
		margin: 0 auto;
		padding: 20px;

		overflow-x: clip;

		// slightly bigger
		font-size: 1.2rem;

		// the code cmd inline elements
		.cmd {

			// dark themed pill
			background: rgba(0, 0, 0, 0.7);			
			padding: 0px 15px;
			border-radius: 50px;

			// text styles
			font-family: monospace;
			font-weight: bolder;
			color: white;
		}// .cmd

		.code-screen{
			box-sizing: content-box;
			border-left: 3px solid #077B95;
			margin-left: 30px;
		}

	}// .overview-box

	.copy-right {
		text-align: center;
		color: white;
	}
	
</style>
