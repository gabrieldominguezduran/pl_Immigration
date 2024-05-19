<script>
	// @ts-nocheck

	import Map from './Map.svelte';
	import Texts from './Texts.svelte';

	let currentYear = '2022';

	function updateYear(event) {
		currentYear = event.detail.year;
	}

	function handleScroll() {
		const sections = document.querySelectorAll('.year-section');
		const scrollPosition = window.scrollY + window.innerHeight / 2;

		for (const section of sections) {
			const { top, bottom } = section.getBoundingClientRect();
			if (top <= scrollPosition && bottom >= scrollPosition) {
				const year = section.dataset.year;
				if (year !== currentYear) {
					currentYear = year;
					break;
				}
			}
		}
	}
</script>

<div
	class="app-container"
	on:scroll={handleScroll}
>
	<div class="map-background">
		<Map currentYear={currentYear} />
	</div>
	<div class="text-container">
		<Texts on:yearChange={updateYear} />
	</div>
</div>

<style>
	.app-container {
		position: relative;
		height: 100vh;
		overflow: auto;
		scroll-behavior: smooth;
		background-color: transparent;
	}

	.map-background {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: -1;
	}

	.text-container {
		width: 100%;
		background-color: transparent;
	}
</style>
