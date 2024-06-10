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

	function handleYearChange(event) {
		onYearChange(event.target.value);
	}

	function handleMouseOver(event, countryName) {
		selectAll('.country-path').attr('opacity', (d) => {
			if (!d || !d.properties) return 0.3;
			return topCountries.find(
				(tc) => tc.properties.ADMIN === d.properties.ADMIN,
			)
				? 1
				: 0.3;
		});
		selectAll('.bar')
			.attr('opacity', (d) => {
				if (!d || !d.properties) return 0.5;
				return d.properties.ADMIN === countryName ? 1 : 0.5;
			})
			.attr('fill', (d) => {
				if (!d || !d.properties) return '#FE3713';
				return d.properties.ADMIN === countryName ? '#FFD700' : '#FE3713';
			});
		selectAll('.country-path')
			.attr('fill', (d) => {
				if (!d || !d.properties) return '#FFFFFF';
				return d.properties.ADMIN === countryName
					? '#FFD700'
					: topCountries.find(
								(tc) => tc.properties.ADMIN === d.properties.ADMIN,
						  )
						? '#FFD700'
						: '#FFFFFF';
			})
			.attr('stroke', (d) => {
				if (!d || !d.properties) return '#444645';
				return d.properties.ADMIN === countryName ? '#FFD700' : '#444645';
			});
	}

	function handleMouseOut() {
		selectAll('.country-path')
			.attr('fill', (d) => {
				if (!d || !d.properties) return '#FFFFFF';
				return topCountries.find(
					(tc) => tc.properties.ADMIN === d.properties.ADMIN,
				)
					? '#FFD700'
					: '#FFFFFF';
			})
			.attr('opacity', 1);
		selectAll('.bar').attr('fill', '#FE3713').attr('opacity', 1);
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
	<svg
		class="bar-chart"
		width={width / 2}
		height={height}
	>
		{#each topCountries as { properties, visaTotal }, i}
			<rect
				class="bar"
				x={0}
				y={i * 22}
				width={visaTotal / 500}
				height={20}
				fill="#FFD700"
				on:mouseover={(event) => handleMouseOver(event, properties.ADMIN)}
				on:mouseout={handleMouseOut}
				role="button"
				tabindex="0"
				on:focus={(event) => handleMouseOver(event, properties.ADMIN)}
				on:blur={handleMouseOut}
			/>
			<text
				x={visaTotal / 500 + 5}
				y={i * 22 + 15}
				fill="white">{properties.ADMIN}: {visaTotal}</text
			>
		{/each}
	</svg>
	<div class="map">
		<svg
			width="100%"
			height="100%"
			viewBox={`0 0 ${width} ${height}`}
			preserveAspectRatio="xMidYMid meet"
		>
			{#each countries as { id, path, properties }}
				<path
					class="country-path"
					d={path}
					fill={topCountries.find(
						(tc) => tc.properties.ADMIN === properties.ADMIN,
					)
						? '#FFD700'
						: '#FFFFFF'}
					stroke="#444645"
					role="button"
					tabindex="0"
					on:mouseover={(event) => handleMouseOver(event, properties.ADMIN)}
					on:mouseout={handleMouseOut}
					on:focus={(event) => handleMouseOver(event, properties.ADMIN)}
					on:blur={handleMouseOut}
				/>
			{/each}
		</svg>
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
	}

	.bar-chart {
		width: 90%;
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

	.bar-chart {
		display: inline-block;
	}

	.bar {
		cursor: pointer;
	}

	path {
		stroke: #444645;
		cursor: pointer;
	}

	text {
		font-size: 0.8rem;
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

		.bar-chart {
			width: 100%;
		}
	}
</style>
