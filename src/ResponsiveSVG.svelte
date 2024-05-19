<script>
	export let width;
	export let height;
</script>

<div
	class="main"
	bind:clientWidth={width}
	bind:clientHeight={height}
>
	<svg
		width={width}
		height={height}
	>
		<slot />
	</svg>
</div>

<style>
	.main {
		width: 100vw;
		height: 60vh;
		margin: 3rem auto;
		overflow: hidden;
	}

	@media (max-width: 600px) {
		.main {
			height: 50vh;
			margin: 0;
		}
	}
</style>
