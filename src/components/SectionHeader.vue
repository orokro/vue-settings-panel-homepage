<!--
	SectionHeader.vue
	-----------------

	Horizontal Separator for the home page sections
-->
<template>
	<div class="outer-border-1">
		<div class="outer-border-2">
			<div class="section-header">
				<div class="section-header-line"></div>
				<h2 class="section-header-text">
					<slot/>
				</h2>
			</div>
		</div>
	</div>
</template>
<script setup>

// vue
import { ref, onMounted } from 'vue';

</script>
<style lang="scss" scoped>
	
	// main row with content
	.section-header {

		/* display: none; */

		// fill page width & provide room above
		width: 100vw;
		
		/* border: 1px solid red; */

		background: rgba(255, 255, 255, 0.25);
		font-size: 30px;
		color: white;
		text-align: center;
		text-shadow: 3px 2px 0px rgba(0, 0, 0, 0.5);
		padding: 5px;

		// well do some cool nesting borders
		border-top: 3px solid white;
		border-bottom: 3px solid white;

		h2 {

			font-weight: bolder;
			text-transform: uppercase;
			font-style: italic;
		}
	
	}//.section-header

	.outer-border-2 {
		padding: 3px 0px;
		border-top: 2px solid white;
		border-bottom: 2px solid white;
	}

	.outer-border-1 {
		padding: 2px 0px;
		border-top: 1px solid white;
		border-bottom: 1px solid white;

		/* box-sizing: content-box; */
		margin-top: 150px;
	}


</style>
