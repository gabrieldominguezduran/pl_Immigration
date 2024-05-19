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
		height: 80vh;
		overflow: hidden;
	}
</style>
