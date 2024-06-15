<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	let dataset = [];
	let xScale;
	let yScale;
	let barColor;
	let xTickValues = [];
	let yTicks = [];

	let width = 950;
	let height = 400;
	const margin = { top: 20, right: 40, bottom: 50, left: 100 };

	const vacanciesData = [
		{ year: '2021', value: 135.975 },
		{ year: '2022', value: 139.8 },
		{ year: '2023', value: 109.1 },
	];

	const unemploymentData = [
		{ year: '2020', value: 5.95 },
		{ year: '2021', value: 6.1 },
		{ year: '2022', value: 5.27 },
	];

	let selectedData = 'vacancies';
	updateChartData();

	onMount(() => {
		updateChartData();
	});

	function updateChartData() {
		dataset = selectedData === 'vacancies' ? vacanciesData : unemploymentData;

		const yMax = selectedData === 'vacancies' ? 150 : 10;
		const yMin = 0;

		xScale = d3
			.scaleBand()
			.domain(dataset.map((d) => d.year))
			.range([0, width - margin.left - margin.right])
			.padding(0.1);

		yScale = d3
			.scaleLinear()
			.domain([yMin, yMax])
			.range([height - margin.top - margin.bottom, 0]);

		barColor =
			selectedData === 'vacancies' ? '#ffd700' : 'rgba(254, 55, 13, 0.8)';

		xTickValues = dataset.map((d) => d.year);
		const division = selectedData === 'vacancies' ? 5 : 5;
		yTicks = d3.range(yMin, yMax, (yMax - yMin) / division).concat([yMax]);
	}

	function toggleData(dataType) {
		selectedData = dataType;
		updateChartData();
	}
</script>

<main class="chart-container">
	<div class="texts">
		<h1>El mercado laboral</h1>
		<p class="description">
			El país enfrenta un notable desafío en su mercado laboral, caracterizado
			por un déficit significativo de trabajadores y fluctuaciones en la tasa de
			desempleo.
		</p>
		<p class="description">
			Los datos recientes muestran variaciones en el número de vacantes de
			empleo, con un promedio anual que aumentó de 136 mil posiciones en 2021 a
			140 mil en 2022, antes de descender a 109 mil en 2023.
		</p>
		<p class="description">
			A pesar de esta fluctuación, la demanda de mano de obra sigue siendo
			fuerte. En paralelo, la tasa de desempleo ha mostrado una tendencia a la
			baja, pasando del 6.1% en 2021 al 5.27% en 2022, indicando un mercado
			laboral dinámico con oportunidades de empleo y una mejora en la situación
			laboral general del país.
		</p>
	</div>
	<div class="chart">
		<div class="buttons">
			<button
				class:selected={selectedData === 'vacancies'}
				on:click={() => toggleData('vacancies')}>Vacantes de empleo</button
			>
			<button
				class:selected={selectedData === 'unemployment'}
				on:click={() => toggleData('unemployment')}>Tasa de desempleo</button
			>
		</div>
		{#if dataset.length === 0}
			<p>Loading...</p>
		{:else}
			<svg
				width="100%"
				height="100%"
				viewBox={`0 0 ${width} ${height}`}
				preserveAspectRatio="xMidYMid meet"
			>
				<g transform={`translate(${margin.left},${margin.top})`}>
					{#each xTickValues as tick}
						<g>
							<text
								x={xScale(tick) + xScale.bandwidth() / 2}
								y={height - margin.bottom}
								dy="1em"
								text-anchor="middle"
								fill="#fff"
							>
								{tick}
							</text>
							<line
								x1={xScale(tick) + xScale.bandwidth() / 2}
								y1={0}
								x2={xScale(tick) + xScale.bandwidth() / 2}
								y2={height - margin.bottom}
								stroke="#333"
								stroke-dasharray="2,2"
							/>
						</g>
					{/each}
					<g>
						<line
							x1={0}
							y1={height - margin.bottom}
							x2={width - margin.left - margin.right}
							y2={height - margin.bottom}
							stroke="#fff"
						/>
						{#each yTicks as tick}
							<g>
								<text
									x={-10}
									y={yScale(tick)}
									dy="0.3em"
									fill="#fff"
									text-anchor="end"
								>
									{selectedData === 'vacancies'
										? tick.toFixed(2)
										: tick.toFixed(2)}
								</text>
								<line
									x1={0}
									y1={yScale(tick)}
									x2={width - margin.left - margin.right}
									y2={yScale(tick)}
									stroke="#333"
									stroke-dasharray="2,2"
								/>
							</g>
						{/each}
						<line
							x1={0}
							y1={0}
							x2={0}
							y2={height - margin.bottom}
							stroke="#fff"
						/>
					</g>
					<g>
						{#each dataset as d}
							<g>
								<rect
									x={xScale(d.year)}
									y={yScale(d.value)}
									width={xScale.bandwidth()}
									height={height - margin.bottom - yScale(d.value)}
									fill={barColor}
								/>
								<text
									x={xScale(d.year) + xScale.bandwidth() / 2}
									y={yScale(d.value) - 10}
									text-anchor="middle"
									fill="white"
									font-size="12px"
								>
									{d.value.toFixed(2)}
								</text>
							</g>
						{/each}
					</g>
				</g>
			</svg>
		{/if}
		<p class="source">
			Fuente: <a
				href="https://dbw.stat.gov.pl/en/dashboard/28"
				target="_blank">https://dbw.stat.gov.pl/en/dashboard/28</a
			>
		</p>
	</div>
</main>

<style>
	.chart-container {
		display: flex;
		justify-content: space-between;
		align-items: center;
		flex-direction: row;
		margin-top: 20px;
	}

	.texts {
		width: 40%;
		margin-left: 3rem;
	}

	.texts h1 {
		font-size: 2rem;
		color: #ffd700;
		margin-bottom: 2rem;
	}

	.description {
		font-size: 1.5rem;
		text-align: left;
		color: #fff;
		padding: 0.5rem 0;
		margin: 0;
	}

	.buttons {
		display: flex;
		justify-content: center;
		margin: 3rem 0;
	}

	button {
		margin: 0 10px;
		padding: 10px 20px;
		font-size: 1rem;
		cursor: pointer;
		border-radius: 10px;
		border: none;
	}

	button:hover {
		opacity: 0.8;
		transform: scale(1.05);
	}

	.selected {
		background-color: #ffd700;
		color: #000;
	}

	.chart {
		width: 55%;
		height: 100%;
	}

	.source {
		text-align: center;
		margin: 1rem 0 3rem;
		font-size: 0.8rem;
		color: #fff;
	}

	.source a {
		color: #ffd700;
		text-decoration: none;
	}

	@media (max-width: 1024px) {
		.chart-container {
			flex-direction: column;
		}

		.texts {
			width: 90%;
			text-align: center;
			margin: 0 auto;
		}

		.chart {
			width: 90%;
			margin-top: 2rem;
		}

		.description {
			font-size: 1.2rem;
			padding: 1rem;
		}
	}
</style>
