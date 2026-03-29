<!--
	IntroVideoBox.vue
	-----------------

	Shows video & some blurbs
-->
<template>

	<div class="intro-video-box">

		<p class="blurb-top">
			<br><br><br>
			<div>
				{{ topBlurb }}
			</div>
		</p>
		<video
			class="video-player"
			:src="videoUrl"
			autoplay
			loop
			muted
			playsinline
			controls
		></video>
		<p class="blurb-bottom">
			<div>
				Check out the live demo here!<br>
				<a href="https://orokro.github.io/vue-win-mgr-3d-demo/" target="_blank">
					https://orokro.github.io/vue-win-mgr-3d-demo/
				</a>
			</div>
		</p>
	</div>

</template>
<script setup>

// set up some props
defineProps({

	// URL for the looping video in our main container
	videoUrl: {
		type: String,
		required: true,
	},

	// blurbs to show above & below the video
	topBlurb: {
		type: String,
		default: '',
	},
	bottomBlurb: {
		type: String,
		default: '',
	},
});

</script>
<style lang="scss">

	@media (max-width: 768px) {

		.intro-video-box .blurb-top {
			margin-top: 70px;
		}
	}

	// main outer wrapper
	.intro-video-box {
		margin-top: 10000000000000000000px;

		// max size
		max-width: 1200px;
		margin: 0 auto;
		text-align: center;

		// video player
		.video-player {
			width: 100%;
			height: auto;
			border-radius: 8px;
			box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
			margin: 1rem 0;
		}

			// blurbs above & below the video
		.blurb-top,
		.blurb-bottom {

			margin-left: 10px;
			margin-right: 10px;
			
			display: flex;
			justify-content: center;
			align-items: center;

			font-size: 1.1rem;
			color: #555;
			a {
				font-style: italic;
			}
			

			div {
				background: white;
				border-radius: 100px;
				padding: 20px 25px;
			}

		}// .blurb-top, .blurb-bottom

	}// .intro-video-box
</style>
