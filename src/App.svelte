<script>
	// @ts-nocheck

	import Map from './Map.svelte';
	import Texts from './Texts.svelte';

	let currentYear = '2018';

	function updateYear(event) {
		currentYear = event.detail.year;
	}

	function handleScroll() {
		const sections = document.querySelectorAll('.year-section');
		const scrollPosition = window.scrollY + window.innerHeight / 2;
		for (const section of sections) {
			const { top, bottom } = section.getBoundingClientRect();
			if (top < scrollPosition && bottom >= scrollPosition) {
				const year = section.dataset.year;
				if (year !== currentYear) {
					currentYear = year;
					break;
				}
			}
		}
	}

	window.addEventListener('scroll', handleScroll);
</script>

<div class="app-container">
	<div class="map-background">
		<Map currentYear={currentYear} />
	</div>
	<div
		class="text-container"
		on:scroll={handleScroll}
	>
		<Texts on:yearChange={updateYear} />
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
	}
</style>
