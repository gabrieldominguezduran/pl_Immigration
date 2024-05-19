<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	let sections;

	function handleScroll() {
		const scrollPosition = window.scrollY + window.innerHeight / 2;

		for (const section of sections.children) {
			const { top, bottom } = section.getBoundingClientRect();
			if (top <= scrollPosition && bottom >= scrollPosition) {
				dispatch('yearChange', { year: section.dataset.year });
				break;
			}
		}
	}
</script>

<div
	class="texts-container"
	bind:this={sections}
	on:scroll={handleScroll}
>
	<section
		class="year-section"
		data-year="2022"
	>
		<div class="text">
			<h2>2022</h2>
			<p>Andrej Duda:</p>
			<blockquote>
				“Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod
				tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
				veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
				commodo consequat.”
			</blockquote>
		</div>
	</section>
	<section class="map-section"></section>
	<!-- Sección vacía para mostrar el mapa -->
	<section
		class="year-section"
		data-year="2023"
	>
		<div class="text">
			<h2>2023</h2>
			<p>Donald Tusk:</p>
			<blockquote>
				“Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod
				tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
				veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
				commodo consequat.”
			</blockquote>
		</div>
	</section>
	<section class="map-section"></section>
	<!-- Sección vacía para mostrar el mapa -->
</div>

<style>
	.texts-container {
		width: 100%;
		background-color: transparent;
	}

	.year-section {
		padding: 20px;
		height: 100vh;
		font-size: 2rem;
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
		justify-content: center;
		background-color: rgba(255, 255, 255, 0.8);
	}

	.map-section {
		height: 100vh;
	}

	blockquote {
		margin: 1.3rem 0;
		color: #555;
	}

	.text {
		max-width: 80%;
		margin: 0 auto;
	}
</style>
