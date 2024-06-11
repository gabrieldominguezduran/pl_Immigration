<script>
	import quotes from '../data/quotes.json';
	import Quote from './Quotes.svelte';

	let groupedQuotes = {};
	for (let year in quotes) {
		groupedQuotes[year] = quotes[year];
	}
</script>

<div class="texts-container">
	<section class="year-section">
		<div class="text">
			{#each Object.keys(groupedQuotes) as year}
				<div class="year-quote">
					<h3>{year}</h3>
					{#each groupedQuotes[year] as quote}
						<div class="quote">
							<Quote
								author={quote.author}
								position={quote.position_spanish}
								quote={quote.quote_spanish}
								source={quote.source}
								source_link={quote.source_link}
							/>
						</div>
					{/each}
				</div>
			{/each}
		</div>
	</section>
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
		font-size: 1.5rem;
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
		justify-content: center;
		background-color: rgba(16, 16, 16, 0.891);
		pointer-events: auto;
	}

	.text {
		display: flex;
		gap: 1rem;
		margin: 0 auto;
		color: #fff;
	}

	.year-quote {
		margin-bottom: 2rem;
	}

	h3 {
		color: #ffd700;
		margin-bottom: 1rem;
	}

	.quote {
		margin-bottom: 1rem;
	}

	@media (max-width: 1200px) {
		.text {
			flex-wrap: wrap;
		}
	}
</style>
