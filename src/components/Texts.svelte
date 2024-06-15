<script>
	import PopulationChart from './PopulationChart.svelte';
	import JobsChart from './JobsChart.svelte';
	import QuotesChart from './QuotesChart.svelte';
</script>

<section class="title-secton">
	<h1>El dilema polaco: cambios en el mercado laboral y la inmigración</h1>
	<div class="photo-credit">
		Photo by <a
			href="https://unsplash.com/@gliwik?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
			target="_blank">Kamil Gliwiński</a
		>
		on
		<a
			href="https://unsplash.com/photos/high-rise-buildings-during-night-time-xcPw1-5OHTk?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
			target="_blank">Unsplash</a
		>
	</div>
</section>
<section class="population-section">
	<PopulationChart />
</section>
<section class="jobs-section">
	<JobsChart />
</section>
<div class="texts-container">
	<section class="quotes-section">
		<QuotesChart />
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

	.title-secton {
		width: 100%;
		height: 80vh;
		display: flex;
		background-image: url('../images/main.jpg');
		background-size: cover;
		background-position: center;
		padding: 1rem;
		box-sizing: border-box;
		position: relative;
	}

	.title-secton h1 {
		width: 50%;
		font-size: 4rem;
		color: #ffd700;
	}
	.photo-credit {
		position: absolute;
		bottom: 10px;
		right: 10px;
		font-size: 0.8rem;
		color: #fff;
		background: rgba(0, 0, 0, 0.5);
		padding: 5px;
		border-radius: 5px;
	}

	.photo-credit a {
		color: #ffd700;
		text-decoration: none;
	}

	.quotes-section {
		padding: 20px;
		font-size: 1.5rem;
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
		justify-content: center;
		background-color: rgba(16, 16, 16, 0.891);
		pointer-events: auto;
	}

	@media (max-width: 1020px) {
		.title-secton {
			min-height: 40vh;
			padding: 0;
		}
		.title-secton h1 {
			font-size: 3rem;
			width: 100%;
			padding: 2rem;
		}
	}
</style>
