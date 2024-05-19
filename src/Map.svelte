<script>
	import { json, geoNaturalEarth1, geoPath, select } from 'd3';
	import ResponsiveSvg from './ResponsiveSVG.svelte';
	import visa2023Data from './data/2023.json';
	import totals from './data/totals.json';

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
		const country = visa2023Data.find((d) => d.Country === countryName);
		return country ? country.Total : null;
	}

	$: projection = geoNaturalEarth1().fitSize([width, height], geojson);
	$: pathGenerator = geoPath(projection);

	let countries = [];
	$: if (geojson)
		countries = geojson.features.map((feature) => {
			return {
				...feature,
				path: pathGenerator(feature),
				visaTotal: getVisaTotal(feature.properties.ADMIN),
			};
		});

	let bubbles = [];
	$: bubbles = countries
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
		.slice(0, 10)
		.map((d, i) => ({ ...d, rank: i + 1 }));

	function handleMouseOver(event) {
		const element = event.currentTarget;
		element.parentNode.appendChild(element); // Move the element to the end of the parent, bringing it to the top
	}

	$: totalVisas = totals.find((total) => total.year === '2023')?.Total || 0;
</script>

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
					r={15 + (8 - rank) * 2}
					fill="rgba(255, 87, 34, 0.8)"
					class="bubble bubble-{id}"
				/>
				<text
					x="0"
					y="0"
					text-anchor="middle"
					fill="#ffffff"
					dy=".3em"
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
					y="0"
					text-anchor="middle"
					fill="#ffffff"
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

<div class="total-visas">
	Total de visas en 2023: {totalVisas}
</div>

<style>
	path {
		fill: rgb(0, 105, 92);
		stroke: #ffffff;
		cursor: pointer;
	}

	.bubble {
		stroke: #ffffff;
		cursor: pointer;
		transition:
			transform 0.3s,
			r 0.3s;
	}

	.bubble-text {
		font: 0.6rem sans-serif;
		pointer-events: none;
		transition: font-size 0.3s;
	}

	.hover-text {
		font: 0.6rem sans-serif;
		pointer-events: none;
		display: none;
	}

	.bubble-group:hover .bubble {
		transform: scale(3);
	}

	.bubble-group:hover .bubble-text {
		display: none;
	}

	.bubble-group:hover .hover-text {
		display: block;
		font-size: 0.8rem;
	}

	.total-visas {
		text-align: center;
		margin-top: 20px;
		font-size: 1.2rem;
	}
</style>
