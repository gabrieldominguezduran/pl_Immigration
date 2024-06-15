<script>
	import { onMount } from 'svelte';
	import { draw } from 'svelte/transition';
	import * as d3 from 'd3';

	let dataset = [];
	let xScale;
	let yScale;
	let line;
	let xTickValues = [];
	let yTicks = [];

	let width = 950;
	let height = 400;
	const margin = { top: 20, right: 40, bottom: 50, left: 100 };

	const populationData = [
		{ year: '2021', value: 37907704.0 },
		{ year: '2022', value: 37766327.0 },
		{ year: '2023', value: 37636508.0 },
	];

	const medianAgeData = [
		{ year: '2021', value: 42.0 },
		{ year: '2022', value: 42.3 },
		{ year: '2023', value: 42.8 },
	];

	let selectedData = 'population';
	updateChartData();

	onMount(() => {
		updateChartData();
	});

	function updateChartData() {
		dataset = selectedData === 'population' ? populationData : medianAgeData;

		const yMax = selectedData === 'population' ? 38000000 : 43;
		const yMin = selectedData === 'population' ? 37000000 : 42;

		xScale = d3
			.scaleBand()
			.domain(dataset.map((d) => d.year))
			.range([0, width - margin.left - margin.right])
			.padding(0.1);

		yScale = d3
			.scaleLinear()
			.domain([yMin, yMax])
			.range([height - margin.top - margin.bottom, 0]);

		line = d3
			.line()
			.x((d) => xScale(d.year) + xScale.bandwidth() / 2)
			.y((d) => yScale(d.value));

		xTickValues = dataset.map((d) => d.year);
		const division = selectedData === 'population' ? 3 : 6;
		yTicks = d3.range(yMin, yMax, (yMax - yMin) / division).concat([yMax]);
	}

	function toggleData(dataType) {
		selectedData = dataType;
		updateChartData();
	}
</script>

<main class="chart-container">
	<div class="texts">
		<h1>Datos demográficos de Polonia</h1>
		<p class="description">
			Polonia enfrenta desafíos demográficos significativos, incluyendo una
			disminución de la población y una baja tasa de natalidad. La emigración ha
			reducido la población activa, creando un vacío en el mercado laboral.
		</p>
	</div>
	<div class="buttons">
		<button
			class:selected={selectedData === 'population'}
			on:click={() => toggleData('population')}>Población total</button
		>
		<button
			class:selected={selectedData === 'age'}
			on:click={() => toggleData('age')}>Edad media</button
		>
	</div>
	{#if dataset.length === 0}
		<p>Loading...</p>
	{:else}
		<svg
			width="90%"
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
								{selectedData === 'population'
									? (tick / 1000000).toFixed(2)
									: tick.toFixed(1)}
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
					<path
						transition:draw={{ duration: 3000 }}
						d={line(dataset)}
						fill="none"
						stroke="yellow"
						stroke-width="2"
					/>
					{#each dataset as d}
						<g>
							<circle
								cx={xScale(d.year) + xScale.bandwidth() / 2}
								cy={yScale(d.value)}
								r="4"
								fill="yellow"
								stroke="#000"
								stroke-width="1"
							/>
							<text
								x={xScale(d.year) + xScale.bandwidth() / 2}
								y={yScale(d.value) - 10}
								text-anchor="middle"
								fill="white"
								font-size="12px"
								opacity="0"
							>
								{selectedData === 'population'
									? d.value.toLocaleString()
									: d.value.toFixed(1)}
							</text>
						</g>
					{/each}
				</g>
				<text
					x={width - margin.right - 100}
					y={margin.top}
					text-anchor="end"
					fill="yellow"
				>
					{selectedData === 'population'
						? `Población: - ${Math.round(populationData[0].value - populationData[2].value).toLocaleString()}`
						: ''}
				</text>
			</g>
		</svg>
	{/if}
	<p class="source">
		Fuente: <a
			href="https://dbw.stat.gov.pl/en/dashboard/21"
			target="_blank">https://dbw.stat.gov.pl/en/dashboard/21</a
		>
	</p>
</main>

<style>
	.chart-container {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		margin-top: 20px;
	}

	.texts {
		width: 60%;
	}

	.texts h1 {
		text-align: center;
		margin-bottom: 2rem;
		font-size: 3rem;
		color: #ffd700;
	}

	.description {
		text-align: left;
		margin-bottom: 2rem;
		font-size: 1.5rem;
		color: #fff;
		padding: 0 8rem;
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

	.source {
		margin: 1rem 0 3rem;
		font-size: 0.8rem;
		color: #fff;
	}

	.source a {
		color: #ffd700;
		text-decoration: none;
	}

	g:hover text {
		opacity: 1;
	}

	@media (max-width: 1024px) {
		.chart-container {
			padding: 0 1rem;
		}

		.texts h1 {
			text-align: left;
			font-size: 2rem;
		}

		.texts {
			width: 90%;
		}

		.description {
			font-size: 1.2rem;
			padding: 0;
		}
	}
</style>
