<!--
	Header.vue
	----------

	Simple header for the page including anchor links that animate to the sections below.
-->
<template>

	<!-- main header box -->
	<div class="header">

		<!-- gradient -->
		<div class="bg-horizontal-gradient"/>
		
		<div class="box logo-box">
			<!-- the entire logo will link to github cuz why not -->
			<a 
				class="github-link"
				href="https://github.com/orokro/Vue-Window-Manager"
				target="_blank"
			>

				<!-- logo -->
				<div class="logo-circle">
					<img 
						class="logo" 
						src="/img/VWM_logo.png"
						alt="Logo"
						height="100%"
					/>
				</div>

				<!-- logo text -->
				<h1 class="logo-text box-bg-color">
					<span>
						vue-win-mgr
					</span>
				</h1>

				<!-- box w/ github logo & text -->
				<div class="github-box">
					<img 
						src="/img/github_icon.png"
						alt="GitHub"
						height="17px"
					/>
					<span>on GitHub</span>
				</div>
			</a>
		</div>

		<!-- links bar -->
		<div class="box nav-box">
			<div
				ref="navItems"
				class="nav-container box-bg-color"
			>
				<div 
					ref="navHighlight"
					class="nav-highlight"
					:style="{
						left: highlightDimensions.left, 
						width: highlightDimensions.width
					}"
				/>

				<!-- loop to generate links -->
				<div 
					v-for="link in links"
					:key="link.slug"
					:class="`nav-item ${link.slug} ${currentLink === link.slug ? 'active' : ''}`"
					@click="gotoLink(link.slug)"
				>
					<span>{{ link.label }}</span>
				</div>
			</div>
		</div>

	</div>

</template>
<script setup>

// vue
import { ref, onMounted, onUnmounted, computed } from 'vue';

// elements
const navItems = ref([]);
const navHighlight = ref(null);

// store currently selected link
const currentLink = ref('demo');

// our link details
const links = [
	{ slug: 'demo', label: 'Demo' },
	{ slug: 'overview', label: 'Overview' },
	{ slug: 'get', label: 'Get' },
	{ slug: 'docs', label: 'Docs' }
];


/**
 * Goes to the link
 * 
 * @param linkSlugName {string} - the slug name of the link to highlight
 */
function gotoLink(linkSlugName) {

	// save our slunk
	// currentLink.value = linkSlugName;
	window.location.hash = `#${linkSlugName}`;
}


// dynamically compute the left/width of the highlight
const highlightDimensions = computed(() => {

	try {
		// get the parent container dom element
		const containerEl = navItems.value;

		// find the DOM element to move to
		const containerRect = containerEl.getBoundingClientRect();
		const targetEl = containerEl.querySelector(`.${currentLink.value}`);
		const targetRect = targetEl.getBoundingClientRect();

		// compute local left and width
		const left = targetRect.left - containerRect.left;
		const width = targetRect.width;

		return {
			left:  `${left}px`,
			width: `${width}px`
		};
	}catch(e){
		return {
			left: '0px',
			width: '0px'
		}
	}
});


// observe the various anchor tags that match our slugs, so we can auto-scroll the header
let observer;
onMounted(() => {

	// observer to watch for sections entering the viewport
	observer = new IntersectionObserver((entries) => {
		entries.forEach(entry => {
			if (entry.isIntersecting) {
			currentLink.value = entry.target.id;
			}
		});
	}, {

		// prevent triggering when the section is only partially visible
		rootMargin: '10% 0px -40% 0px',

		// Trigger when 100% of the section is visible
		threshold: 1,
	});

	// observe each of the anchor tags w/ matching slug
	links.forEach(link => {
		const slug = link.slug;
		const el = document.getElementById(slug);
		if (el)
			observer.observe(el);
	});
});


// clean up listener, even though we'll probably never unmount the header
onUnmounted(() => {

	if (observer)
		observer.disconnect();
});


</script>
<style lang="scss" scoped>

	// outer most box
	.header {

		// disable text highlighting
		user-select: none;

		// fixed on top
		position: fixed;
		inset: 0px 0px auto 0px;
		min-height: 80px;
		width: 100vw;

		// dark blur background
		background: rgba(255, 255, 255, 0.6);
		backdrop-filter: blur(15px);

		border-bottom: 3px solid white;

		// gradient background
		.bg-horizontal-gradient {

			// fill container, same as .header
			position: absolute;
			inset: 0px 0px 0px 0px;

		}// .bg-horizontal-gradient

		// common bg color
		.box-bg-color {
			background: rgba(255, 255, 255, .9);
		}

		// stack stuff left
		display: flex;
		flex-direction: row;
		justify-content: center;
		flex-wrap: wrap;
		align-items: end;

		// generic styles for the boxes in our flex layout
		.box {
			/* border: 1px solid red; */
			position: relative;
			flex-grow: 0;
			flex-shrink: 0;
			flex-basis: auto;		
		}// box

		// flex box for header logo
		.logo-box {

			// fixed size
			width: 360px;
			height: 80px;

			/* border: 1px solid red; */

			// fixed on top-left
			.logo-circle {

				// fixed on top-left
				position: absolute;
				bottom: 7px;
				left: 0px;
				
				// black circle
				height: 65px;
				

				// logo image
				.logo {
					
					// don't allow it to be dragged
					user-select: none;
					-webkit-user-drag: none;
					
				}// .logo

			}// .logo-circle

			// text that says "ThreeQuery" next to the logo
			.logo-text {

				// fixed position next to logo image
				position: absolute;
				bottom: 7px;
				left: 150px;
				height: 40px;

				// pretty corners
				border-radius: 5px 5px 40px 5px;

				// spacing
				padding: 0px 20px 0px 5px;

				// text styles
				color: #7d50f8;
				font-weight: bolder;
				font-style: italic;

				span {
					position: relative;
					top: -6px;
					font-weight: inherit;
				}

			}// .logo-text

			// clickable box to goto git hob, above / part of the logo
			.github-box {

				// for debug
				/* border: 1px solid red; */

				// fixed above the logo
				position: absolute;
				top: 12px;
				left: 150px;
				height: 22px;
				width: 120px;

				// padding for logo on left
				padding: 0px 0px 0px 21px;

				color: #EFEFEF;
				

				// the logo image
				img {
					position: absolute;
					top: 0px;
					left: 0px;
				}

				span {
					font-weight: inherit;
					position: relative;
					top: -4px;
					color: #7d50f8
				}

				&:hover {
					color: white;
					font-weight: bolder;
				}

			}// .github-box

		}// .logo-box

		// nav bar box
		.nav-box {

			width: 335px;
			height: 50px;

			// nav bar links flex layout
			.nav-container {

				// fixed near the logo
				position: absolute;
				bottom: 8px;
				left: 0px;
				height: 40px;

				// fancy styles
				border-radius: 40px 5px;

				display: flex;
				overflow: hidden;

				/* background: #f0f0f0; */
				padding: 10px 0;
				justify-content: center;

				// nav buttons
				.nav-item {

					// layout & spacing
					padding: 0px 20px;
					position: relative;
					z-index: 1;


					&:nth-child(2) {
						padding-left: 20px;
					}
					&:last-child {
						padding-right: 25px;
					}

					// appear clickable
					cursor: pointer;
					
					// text styles
					color: #8860f7;
					font-weight: bold;
					transition: color 0.3s ease;

					span {
						position: relative;
						top: -3px;
						font-weight: inherit;
					}// span

					&:hover {
						color: #00ABAE;
						font-weight: bolder;
					}
					
					&.active {
						color: white;
						font-weight: bolder;
					}
					
				}// .nav-item
				
				// highlight to slide left & right when nave selected
				.nav-highlight {

					// fixed bos
					position: absolute;

					// fill box
					top: 0px;
					bottom: 0px;
					left: 0px;
					width: 82.5px;

					// nice teal styles
					background: #7d50f8;

					// same shape as .nav-container
					border-radius: 40px 5px;

					transition: all 0.3s ease;
					z-index: 0;

				}// .nav-highlight
				
			}// .nav-container

		}// .nav-box

	}// .header
	
</style>
