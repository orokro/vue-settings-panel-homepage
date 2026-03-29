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
		top-blurb="Check out the demo reel below & scroll down to learn how to use vue-win-mgr!"
	/>

	<!-- overview video -->
	<a id="overview"/>
	<SectionHeader>Overview</SectionHeader>
	<br><br>
	<div class="overview-box">

		<div align="center">
			<img class="logo" src="/img/VWM_logo.png" alt="Vue Win Mgr Logo" width="200px"/>
		</div>

		<h2>vue-win-mgr In a Nutshell:</h2>
		<br>
		<p>
			The <code>vue-win-mgr</code> library is easy to implement. It just uses regular ol' Vue components as it's windows.
			They don't need to meet any specific requirements. You can use <em>any</em> state solution you want for your app,
			just think of the windows as Vue components that can be moved around, docked, snapped, resized, closed, etc.
		</p>
		<br/>
		<p>Just import a bunch of components to use as windows:</p>
		<br>
		<img src="/img/overview_00_regular_components.png" width="100%" class="code-screen" style="max-width: 778px;"/>

		<br><br><br><br>
		<p>
			Next, prepare the windows in an Array, by providing:
		</p>
		<ul>
			<li><strong>window</strong> &mdash; the imported Vue component constructor</li>
			<li><strong>title</strong> &mdash; a title for the window when instantiated (this can be programmatically changed as well)</li>
			<li><strong>slug</strong> &mdash; a string we can refer to this window kind in layouts or code</li>
			<li><strong>icon</strong> &mdash; and optional path to an image to use as the window icon</li>
		</ul>
		<br>
		<p>
			See the example here:
		</p>
		<br>
		<img src="/img/overview_10_arrray_of_windows.png" width="100%" class="code-screen" style="max-width: 527px;"/>

		<br><br><br><br>

		<p>
			If you want, you can also use the window managers built in header (top bar) and status bar system.
			<br><br>
			Let's bring in some components:
		</p>
		<br>
		<img src="/img/overview_20_headerbar_and_statusbar_components_as_well.png" width="100%" class="code-screen" style="max-width: 487px;"/>

		<br><br><br><br>

		<p>
			Now let's put it all together!
			<br><br>
			The snippet below is a pretty self-explanatory example of the <code>&lt;WindowManager\&gt;</code> component
			implemented in a template.
			<br><br>
			You can check out the layout field in the docs below for a full breakdown (it's easy)
		</p>
		<br>
		<img src="/img/overview_30_implementing_the_component.png" width="100%" class="code-screen" style="max-width: 969px;"/>

		<br><br><br><br>

		<p>
			If you don't want to make components for the header and status bar, you can use named slots
			as well, consider the example below:
		</p>
		<br>
		<img src="/img/overview_40_named_slots_alternative.png" width="100%" class="code-screen" style="max-width: 726px;"/>

		<br><br><br><br>

		<p>
			That's pretty much it! <br>
			Of course, there's plenty more to discuss, but that's the gist of it.
			<br><br>
			You can also programmatically interact with the system via three provided contexts:
		</p>
		<ul>
			<li>
				<strong>WindowManagerContext</strong> &mdash; 
				Allows some global settings to be changed programmatically &amp; layouts to be exported/loaded.
			</li>
			<li>
				<strong>WindowFrameContext</strong> &mdash; 
				Windows can interact with the WindowFrame they're docked in programmatically.
			</li>
			<li>
				<strong>WindowContext</strong> &mdash; 
				Allows your window components to manipulate their actual window instance, like setting title programmatically.
			</li>
		</ul>
		<br>
		<p>
			For the window manager you can just add a ref to the component, like so
		</p>
		<img src="/img/overview_50_add_ref_to_mananger.png" width="100%" class="code-screen" style="max-width: 463px;"/>
		<br>
		<p>
			You can then call <code>.getContext()</code> on the component reference to get the <code>WindowManagerContext</code>
		</p>
		<br>
		<p>
			For the others, you can <code>inject</code> them into your components, like so:
		</p>
		<img src="/img/overview_60_geting_contexts.png" width="100%" class="code-screen" style="max-width: 577px;"/>

		<br><br>


		<h2>vue-win-mgr Tutorial:</h2>
		<br>
		<p>
			Below is a full tutorial for how to get started with with the <code>vue-win-mgr</code>!
		</p>
		<br>
		<div align="center" style="min-height: 60vh;">
			<iframe
				width="100%"
				style="width: 100%; aspect-ratio: 16/9; max-width: 1200px;"
				src="https://www.youtube.com/embed/h8QUzUtAqDI?si=T5IAY3ZcZGBOPbHv"
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
		<pre class="code">npm install vue-win-mgr</pre>
		Then...
		<pre class="code">import { WindowManager, FRAME_STYLE } from 'vue-win-mgr';</pre>
		Or get everything:
		<pre class="code">
import { 
	WindowManager, 
	FRAME_STYLE,
	WindowManagerContext,
	WindowFrameContext,
	WindowContext
} from 'vue-win-mgr';
		</pre>

		<br><br>
		<h2>Check out vue-win-mgr over here:</h2>
		<div class="row">

			<div class="box">
				<div>
					Find it on
					<a href="https://www.npmjs.com/package/vue-win-mgr" target="_blank">NPM:</a>
				</div>
				<a href="https://www.npmjs.com/package/vue-win-mgr" target="_blank">
					<img class="site-logo" src="/img/npm_logo.png" alt="NPM Logo" height="70px"/>
				</a>
			</div>

			<div class="box">
				<div>
					Find it on
					<a href="https://github.com/orokro/Vue-Window-Manager" target="_blank">GitHub:</a>
				</div>
				<a href="https://github.com/orokro/Vue-Window-Manager" target="_blank">
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
			background: linear-gradient(#b0e9fa, #695675);

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
			border-left: 3px solid #7f49a0;
			margin-left: 30px;
		}

	}// .overview-box

	.copy-right {
		text-align: center;
		color: white;
	}
	
</style>
