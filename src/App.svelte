<script>
	// @ts-nocheck

	import Map from './components/Map.svelte';
	import Texts from './components/Texts.svelte';

	let currentYear = '2018';

	function handleYearChange(newYear) {
		currentYear = newYear;
	}
</script>

<div class="app-container">
	<div class="text-container">
		<Texts />
	</div>
	<div class="map-background">
		<Map
			currentYear={currentYear}
			onYearChange={handleYearChange}
		/>
	</div>
</div>

<style>
	.app-container {
		position: relative;
		height: 100vh;
		scroll-behavior: smooth;
	}

	.map-background {
		position: relative;
		width: 100%;
		height: auto;
		margin-bottom: 2rem;
		padding: 1rem;
	}

	.text-container {
		position: relative;
		width: 100%;
		pointer-events: none;
		color-scheme: dark;
		padding: 1rem;
	}
</style>
