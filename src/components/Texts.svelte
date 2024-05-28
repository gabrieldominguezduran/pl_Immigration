<script>
	import { onMount } from 'svelte';
	import Quote from './Quotes.svelte';
	import quotes from '../data/quotes.json';
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

	let groupedQuotes = {};
	for (let year in quotes) {
		groupedQuotes[year] = quotes[year];
	}
</script>

<div class="texts-container">
	{#each Object.keys(groupedQuotes) as year}
		<section
			class="year-section"
			data-year={year}
		>
			<div class="text">
				<h2>{year}</h2>
				<div class="quote-container">
					{#each groupedQuotes[year] as quote}
						<div class="quote">
							<Quote
								author={quote.author}
								position={quote.position}
								quote={quote.quote_english}
								source={quote.source}
								source_link={quote.source_link}
							/>
						</div>
					{/each}
				</div>
			</div>
		</section>
		<section class="map-section"></section>
	{/each}
</div>

<style>
	.texts-container {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 20px;
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

	.text {
		max-width: 80%;
		margin: 0 auto;
		color: #fff;
	}

	h2 {
		text-align: center;
		color: #ffd700;
		margin-bottom: 2rem;
	}

	.quote-container {
		display: flex;
		justify-content: space-between;
	}

	.quote {
		flex: 1;
		margin: 0 1rem;
	}
</style>
