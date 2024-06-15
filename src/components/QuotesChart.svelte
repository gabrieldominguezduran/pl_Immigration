<script>
	import quotes from '../data/quotes.json';
	import Quote from './Quotes.svelte';

	let groupedQuotes = {};
	for (let year in quotes) {
		groupedQuotes[year] = quotes[year];
	}
</script>

<div class="texts">
	<h1>El discurso recurrente de algunos políticos</h1>
	<p class="description">
		En los últimos cinco años, varios políticos polacos han adoptado una postura
		firmemente antiinmigración, reflejando una retórica que resuena con una
		parte significativa de la población.
	</p>
</div>
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

<style>
	.texts h1 {
		text-align: center;
		margin-bottom: 2rem;
		font-size: 3rem;
		color: #ffd700;
	}

	.description {
		text-align: left;
		margin-bottom: 5rem;
		font-size: 1.5rem;
		color: #fff;
		padding: 0 8rem;
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

	@media (max-width: 1024px) {
		.texts h1 {
			text-align: left;
			font-size: 2rem;
		}

		.description {
			font-size: 1.2rem;
			padding: 0 1rem;
		}

		.text {
			flex-direction: column;
		}
	}
</style>
