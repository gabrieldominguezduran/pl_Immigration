<script>
	import Map from './Map.svelte';
	import Texts from './Texts.svelte';

	let currentYear = 2023;

	function updateYear(newYear) {
		currentYear = newYear;
	}
</script>

<div class="container">
	<div class="text-container">
		<Texts />
	</div>
	<div class="map-container">
		<Map />
	</div>
</div>

<style>
	.container {
		display: flex;
		flex-direction: row;
		align-items: stretch;
		height: 100vh;
		overflow: hidden;
	}

	.text-container {
		width: 30%;
		overflow-y: auto;
		background: #f0f0f0;
	}

	.map-container {
		width: 70%;
		display: flex;
		justify-content: center;
		align-items: center;
		background: #ffffff;
	}
</style>
