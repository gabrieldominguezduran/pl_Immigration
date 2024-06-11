<script>
	import { json, geoNaturalEarth1, geoPath, selectAll } from 'd3';
	import visasData from '../data/visas.json';
	import totals from '../data/totals.json';

	export let currentYear;
	export let onYearChange;
	let width = 800;
	let height = 500;
	const geojsonPath =
		'https://raw.githubusercontent.com/datasets/geo-countries/master/data/countries.geojson';
	const unwantedCountries = ['Antarctica', 'Greenland'];
	let geojson;

	json(geojsonPath).then((data) => (geojson = filterGeoJSON(data)));

	function filterGeoJSON(data) {
		return {
			...data,
			features: data.features.filter(
				(feature) => !unwantedCountries.includes(feature.properties.ADMIN),
			),
		};
	}

	function getVisaTotal(countryName) {
		const yearData = visasData[currentYear] || [];
		const country = yearData.find((d) => d.Country === countryName);
		return country ? country.Total : 0;
	}

	$: projection = geoNaturalEarth1().fitSize([width, height], geojson);
	$: pathGenerator = geoPath(projection);

	let countries = [];
	$: if (geojson) {
		countries = geojson.features.map((feature) => {
			return {
				...feature,
				path: pathGenerator(feature),
				visaTotal: getVisaTotal(feature.properties.ADMIN),
			};
		});
	}

	let topCountries = [];
	$: {
		if (geojson && currentYear) {
			topCountries = countries
				.map((country) => ({
					...country,
					visaTotal: getVisaTotal(country.properties.ADMIN),
				}))
				.filter((d) => d.visaTotal > 0)
				.sort((a, b) => b.visaTotal - a.visaTotal)
				.slice(0, 20);
		}
	}

	let bubbles = [];
	$: if (topCountries.length > 0) {
		bubbles = topCountries.map((country, i) => {
			const centroid = pathGenerator.centroid(country);
			return {
				...country,
				countryName: country.properties.ADMIN,
				visaTotal: country.visaTotal,
				centroid,
				id: country.id,
				rank: i + 1,
			};
		});
	}

	let polandCentroid;
	$: {
		const poland = countries.find(
			(country) => country.properties.ADMIN === 'Poland',
		);
		if (poland) {
			polandCentroid = pathGenerator.centroid(poland);
		}
	}

	function handleYearChange(event) {
		onYearChange(event.target.value);
	}

	function handleMouseOver(event, countryCentroid) {
		const element = event.currentTarget;
		element.parentNode.appendChild(element);

		if (polandCentroid) {
			const svg = selectAll('svg');
			svg
				.append('line')
				.attr('class', 'arrow-line')
				.attr('x1', countryCentroid[0])
				.attr('y1', countryCentroid[1])
				.attr('x2', polandCentroid[0])
				.attr('y2', polandCentroid[1])
				.attr('stroke', '#77F13D')
				.attr('stroke-width', 1)
				.attr('marker-end', 'url(#arrow)')
				.style('animation', 'draw 2s linear forwards');
		}
	}

	function handleMouseOut(event) {
		selectAll('.arrow-line').remove();
	}

	let firstPart = [];
	let secondPart = [];
	let thirdPart = [];
	let fourPart = [];
	$: if (topCountries.length > 0) {
		firstPart = topCountries.slice(0, 5);
		secondPart = topCountries.slice(5, 10);
		thirdPart = topCountries.slice(10, 15);
		fourPart = topCountries.slice(15, 20);
	}
</script>

<div class="title">
	<h1>Los 20 países con más visas en el año {currentYear}</h1>
</div>

<div class="controls">
	<label for="year">Select Year:</label>
	<select
		id="year"
		on:change={handleYearChange}
	>
		{#each Object.keys(visasData) as year}
			<option
				value={year}
				selected={year === currentYear}>{year}</option
			>
		{/each}
	</select>
</div>

<div class="visualization">
	<div class="map">
		<svg
			width="100%"
			height="50%"
			viewBox={`0 0 ${width} ${height}`}
			preserveAspectRatio="xMidYMid meet"
		>
			<defs>
				<marker
					id="arrow"
					markerWidth="8"
					markerHeight="8"
					refX="5"
					refY="5"
					orient="auto"
					markerUnits="strokeWidth"
				>
					<path
						d="M0,0 L0,10 L10,5 z"
						fill="#77F13D"
					/>
				</marker>
			</defs>

			{#each countries as { id, path, properties }}
				<path
					class="country-path"
					d={path}
					fill={properties.ADMIN === 'Poland'
						? '#F0350D'
						: topCountries.find(
									(tc) => tc.properties.ADMIN === properties.ADMIN,
							  )
							? '#FFD700'
							: '#FFFFFF'}
					stroke="#444645"
					role="button"
					tabindex="0"
				/>
			{/each}

			{#each bubbles as { countryName, visaTotal, centroid, id, rank }}
				{#if centroid}
					<g
						transform={`translate(${centroid[0]}, ${centroid[1]})`}
						class="bubble-group"
						role="button"
						tabindex="0"
						on:mouseover={(event) => handleMouseOver(event, centroid)}
						on:mouseout={handleMouseOut}
						on:focus={(event) => handleMouseOver(event, centroid)}
						on:blur={handleMouseOut}
					>
						<circle
							r={8}
							fill="rgba(254, 55, 13, 0.8)"
							class="bubble bubble-{id}"
						/>
						<text
							x="0"
							y="7"
							text-anchor="middle"
							fill="#000"
							dy="1rem"
							class="bubble-text bubble-text-{id}"
						>
							<tspan
								x="0"
								dy="-.6em"
								font-size="0.7em">{rank}</tspan
							>
						</text>
						<text
							x="0"
							y={-8}
							text-anchor="middle"
							fill="#D4FE0D"
							dy="0"
							class="hover-text"
						>
							<tspan
								x="0"
								dy="-.6em"
								font-size="0.7em">{rank}</tspan
							>
							<tspan
								x="0"
								dy="1.2em">{countryName}</tspan
							>
							<tspan
								x="0"
								dy="1.2em">Visas: {visaTotal}</tspan
							>
						</text>
					</g>
				{/if}
			{/each}
		</svg>
	</div>
</div>

<div class="country-names">
	<div class="first-part">
		{#each firstPart as { properties, visaTotal }, i}
			<div class="country">
				{#if i < 10}
					<span>{i + 1}. {properties.ADMIN}: {visaTotal}</span>
				{/if}
			</div>
		{/each}
	</div>
	<div class="second-part">
		{#each secondPart as { properties, visaTotal }, i}
			<div class="country">
				<span>{i + 6}. {properties.ADMIN}: {visaTotal}</span>
			</div>
		{/each}
	</div>
	<div class="third-part">
		{#each thirdPart as { properties, visaTotal }, i}
			<div class="country">
				<span>{i + 11}. {properties.ADMIN}: {visaTotal}</span>
			</div>
		{/each}
	</div>
	<div class="four-part">
		{#each fourPart as { properties, visaTotal }, i}
			<div class="country">
				<span>{i + 16}. {properties.ADMIN}: {visaTotal}</span>
			</div>
		{/each}
	</div>
</div>

<div class="total-visas">
	Total de visas en {currentYear}: {totals.find(
		(total) => total.year === currentYear,
	)?.Total || 0}
</div>

<style>
	.title {
		text-align: center;
		margin-top: 1rem;
		font-size: 1.5rem;
		color: #ffd700;
	}

	.controls {
		text-align: center;
		margin-top: 1rem;
		color: #ffd700;
	}

	.map {
		width: 100%;
		height: 50%;
	}

	.total-visas {
		text-align: center;
		margin-top: 1rem;
		font-size: 1.2rem;
		color: #ffd700;
	}

	.visualization {
		display: flex;
		justify-content: space-around;
		align-items: center;
		margin-top: 1rem;
	}

	.country-names {
		display: flex;
		justify-content: space-around;
		margin: 1rem auto;
	}

	.first-part,
	.second-part,
	.third-part,
	.four-part {
		width: 23%;
		text-align: left;
	}

	.country {
		font-size: 1.2rem;
		color: #ffd700;
		margin-bottom: 0.5rem;
		cursor: pointer;
	}

	path {
		stroke: #444645;
		cursor: pointer;
	}

	.bubble {
		cursor: pointer;
		transition:
			transform 0.3s,
			r 0.3s;
	}

	.bubble-text {
		font: 0.8rem sans-serif;
		pointer-events: none;
		transition: font-size 0.3s;
	}

	.hover-text {
		font: 1rem sans-serif;
		pointer-events: none;
		display: none;
	}

	.bubble-group:hover .bubble {
		transform: scale(8);
	}

	.bubble-group:hover .bubble-text {
		display: none;
	}

	.bubble-group:hover .hover-text {
		display: block;
		font-size: 0.8rem;
	}

	.arrow-line {
		stroke: rgba(244, 81, 81, 0.891);
		stroke-width: 2px;
		marker-end: url(#arrow);
	}

	@keyframes draw {
		from {
			stroke-dasharray: 0, 1000;
		}
		to {
			stroke-dasharray: 1000, 0;
		}
	}

	@media (max-width: 1200px) {
		.visualization {
			flex-direction: column-reverse;
		}

		.title {
			font-size: 1.2rem;
		}

		.total-visas {
			font-size: 1rem;
		}
	}

	@media (max-width: 600px) {
		.map {
			width: 100%;
		}

		.country-names {
			flex-direction: column;
			align-items: center;
		}

		.first-part,
		.second-part,
		.third-part,
		.four-part {
			width: 50%;
			text-align: left;
		}
	}
</style>
