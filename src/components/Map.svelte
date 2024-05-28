<script>
	import { json, geoNaturalEarth1, geoPath } from 'd3';
	import ResponsiveSvg from './ResponsiveSVG.svelte';
	import visasData from '../data/visas.json';
	import totals from '../data/totals.json';

	export let currentYear;
	let numOfCountriesOptions = [10, 15, 20];
	let numOfCountries = numOfCountriesOptions.at(0);

	let width, height;

	const geojsonPath =
		'https://raw.githubusercontent.com/datasets/geo-countries/master/data/countries.geojson';
	const unwantedCountries = ['Antarctica', 'Greenland'];

	function filterGeoJSON(data) {
		return {
			...data,
			features: data.features.filter(
				(feature) => !unwantedCountries.includes(feature.properties.ADMIN),
			),
		};
	}

	let geojson;
	json(geojsonPath).then((data) => (geojson = filterGeoJSON(data)));

	function getVisaTotal(countryName) {
		const yearData = visasData[currentYear] || [];
		const country = yearData.find((d) => d.Country === countryName);
		return country ? country.Total : null;
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

	let bubbles = [];
	$: {
		if (geojson && currentYear) {
			const countryData = countries
				.map((country) => {
					const visaTotal = getVisaTotal(country.properties.ADMIN);
					if (visaTotal !== null) {
						const centroid = pathGenerator.centroid(country);
						return {
							countryName: country.properties.ADMIN,
							visaTotal,
							centroid,
							id: country.id,
						};
					}
					return null;
				})
				.filter((d) => d !== null)
				.sort((a, b) => b.visaTotal - a.visaTotal)
				.slice(0, numOfCountries);

			bubbles = countryData.map((d, i) => ({ ...d, rank: i + 1 }));
		}
	}

	function handleMouseOver(event) {
		const element = event.currentTarget;
		element.parentNode.appendChild(element);
	}

	$: totalVisas =
		(totals.find((total) => total.year === currentYear) || {}).Total || 0;

	function handleNumOfCountriesChange(event) {
		numOfCountries = parseInt(event.target.value);
	}
</script>

<div class="title">
	<h1>
		Los {`${numOfCountries}`} países con más visas en el año {currentYear}
	</h1>
</div>

<ResponsiveSvg
	bind:width={width}
	bind:height={height}
>
	{#each countries as { id, path, properties }}
		<path
			d={path}
			role="button"
			tabindex="0"
		/>
	{/each}

	{#each bubbles as { countryName, visaTotal, centroid, id, rank }}
		{#if centroid}
			<g
				transform={`translate(${centroid[0]}, ${centroid[1]})`}
				class="bubble-group"
				on:mouseover={handleMouseOver}
				role="button"
				tabindex="0"
				on:focus={handleMouseOver}
			>
				<circle
					r={10 + (numOfCountries - rank) * 1}
					fill="rgba(254, 55, 13, 0.8)"
					class="bubble bubble-{id}"
				/>
				<text
					x="0"
					y="7"
					text-anchor="middle"
					fill="#D4FE0D"
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
</ResponsiveSvg>

<div class="num-of-countries-selector">
	<label for="numOfCountries">Número de países:</label>
	{#each numOfCountriesOptions as option}
		<label>
			<input
				type="radio"
				name="numOfCountries"
				value={option}
				bind:group={numOfCountries}
				on:change={handleNumOfCountriesChange}
				checked={numOfCountries === option}
			/>
			{option}
		</label>
	{/each}
</div>

<div class="total-visas">
	Total de visas en {currentYear}: {totalVisas}
</div>

<style>
	.title {
		text-align: center;
		margin-top: 3rem;
		font-size: 1.5rem;
	}

	.total-visas {
		text-align: center;
		margin-top: 20px;
		font-size: 1.2rem;
	}

	path {
		fill: rgb(218, 247, 166);
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
		transform: scale(4);
	}

	.bubble-group:hover .bubble-text {
		display: none;
	}

	.bubble-group:hover .hover-text {
		display: block;
		font-size: 0.8rem;
	}

	.num-of-countries-selector {
		text-align: center;
		margin-top: 1rem;
	}

	.num-of-countries-selector label {
		margin: 0 0.3rem;
		font-size: 1rem;
	}

	.num-of-countries-selector input {
		margin-right: 0.3rem;
	}

	@media (max-width: 600px) {
		.title {
			font-size: 1.2rem;
		}

		.bubble-text {
			font-size: 0.6rem;
		}
	}
</style>
