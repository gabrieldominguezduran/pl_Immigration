<script>
	import { onMount } from 'svelte';

	export let onYearChange;

	let yearSections;

	onMount(() => {
		yearSections = document.querySelectorAll('.year-section');

		const options = {
			root: null,
			rootMargin: '0px',
			threshold: 0.6, // Adjust this value based on when you want to trigger the change
		};

		const callback = (entries) => {
			entries.forEach((entry) => {
				if (entry.isIntersecting) {
					const year = entry.target.getAttribute('data-year');
					onYearChange(year);
				}
			});
		};

		const observer = new IntersectionObserver(callback, options);

		yearSections.forEach((section) => observer.observe(section));

		return () => {
			yearSections.forEach((section) => observer.unobserve(section));
		};
	});
</script>

<div class="texts-container">
	<section
		class="year-section"
		data-year="2018"
	>
		<div class="text">
			<h2>2018</h2>
			<p>Jhon Doe:</p>
			<blockquote>
				“Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod
				tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
				veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
				commodo consequat.”
			</blockquote>
		</div>
	</section>
	<section class="map-section"></section>
	<section
		class="year-section"
		data-year="2019"
	>
		<div class="text">
			<h2>2019</h2>
			<p>Jhon Doe:</p>
			<blockquote>
				“Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod
				tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
				veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
				commodo consequat.”
			</blockquote>
		</div>
	</section>
	<section class="map-section"></section>
	<section
		class="year-section"
		data-year="2020"
	>
		<div class="text">
			<h2>2020</h2>
			<p>Jhon Doe:</p>
			<blockquote>
				“Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod
				tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
				veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
				commodo consequat.”
			</blockquote>
		</div>
	</section>
	<section class="map-section"></section>
	<section
		class="year-section"
		data-year="2021"
	>
		<div class="text">
			<h2>2021</h2>
			<p>Jhon Doe:</p>
			<blockquote>
				“Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod
				tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
				veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
				commodo consequat.”
			</blockquote>
		</div>
	</section>
	<section class="map-section"></section>
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
</div>

<style>
	.texts-container {
		width: 100%;
	}

	.year-section {
		padding: 20px;
		height: 100vh;
		font-size: 2rem;
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
		justify-content: center;
		background-color: rgba(16, 16, 16, 0.891);
		pointer-events: auto;
	}

	.map-section {
		height: 100vh;
	}

	blockquote {
		margin: 1.3rem 0;
		color: #c5c3c3;
	}

	.text {
		max-width: 80%;
		margin: 0 auto;
	}
</style>
