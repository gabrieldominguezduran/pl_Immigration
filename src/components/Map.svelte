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

<div class="texts">
	<h1>Crecimiento de la inmigración</h1>
	<p class="description">
		Se ha experimentado un aumento significativo en el número de inmigrantes,
		con un notable incremento en los dos últimos años de personas provenientes
		de Asia y Sudamérica.
	</p>
	<p class="description">
		Polonia es el país de la Unión Europea que más visas de trabajo ha otorgado
		a ciudadanos de países no pertenecientes a la Unión Europea en los últimos
		años.
	</p>
	<p class="source-text">
		Fuente: <a
			href="https://ec.europa.eu/eurostat/databrowser/view/MIGR_RESFIRST__custom_11823928/default/bar?lang=en&page=time:2022"
			target="_blank"
			>https://ec.europa.eu/eurostat/databrowser/view/MIGR_RESFIRST__custom_11823928/default/bar?lang=en&page=time:2022</a
		>
	</p>
	<div class="subtitle">
		<h2>Los 20 países con más visas en el año {currentYear}</h2>
	</div>
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
						on:focus={(event) => handleMouseOver(event, centroid)}
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
<p class="source">
	Fuente: <a
		href=" https://psz.praca.gov.pl/web/urzad-pracy/-/8180075-zezwolenia-na-prace-cudzoziemcow "
		target="_blank"
	>
		https://psz.praca.gov.pl/web/urzad-pracy/-/8180075-zezwolenia-na-prace-cudzoziemcow
	</a>
</p>

<style>
	.texts {
		text-align: center;
		width: 60%;
		margin: 2rem auto;
	}

	.texts h1 {
		font-size: 3rem;
		color: #ffd700;
		margin-bottom: 2rem;
	}

	.description {
		font-size: 1.5rem;
		text-align: left;
		color: #fff;
		padding: 0.5rem;
		margin: 0;
	}
	.subtitle {
		text-align: center;
		margin: 2rem 0;
		font-size: 1.5rem;
		color: #ffd700;
	}

	.controls {
		text-align: center;
		margin-top: 1rem;
		color: #fff;
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
		color: #fff;
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

	.source-text {
		text-align: left;
		margin: 1rem 0 3rem;
		font-size: 0.8rem;
		color: #fff;
		padding: 0.5rem;
	}

	.source-text a {
		color: #ffd700;
		text-decoration: none;
		line-break: anywhere;
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

	@media (max-width: 1024px) {
		.texts {
			width: 90%;
		}
		.texts h1 {
			text-align: left;
			font-size: 2rem;
		}

		.subtitle {
			font-size: 1.2rem;
		}

		.description {
			font-size: 1.2rem;
			padding: 0;
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
