<script>
	// @ts-nocheck

	import Map from './Map.svelte';
	import Texts from './Texts.svelte';

	let currentYear = '2018';

	function handleYearChange(newYear) {
		currentYear = newYear;
	}
</script>

<div class="app-container">
	<div class="map-background">
		<Map currentYear={currentYear} />
	</div>
	<div class="text-container">
		<Texts onYearChange={handleYearChange} />
	</div>
</div>

<style>
	.app-container {
		position: relative;
		height: 100vh;
		scroll-behavior: smooth;
	}

	.map-background {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	.text-container {
		position: relative;
		width: 100%;
		pointer-events: none;
		color-scheme: dark;
	}
</style>
