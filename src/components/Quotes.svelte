<script>
	export let author;
	export let position;
	export let quote;
	export let source;
	export let source_link;
</script>

<div class="quote-block">
	<p><strong>{author}</strong></p>
	<blockquote>"{quote}"</blockquote>
	<span class="position">{position}</span>
	<p class="source">
		<a
			href={source_link}
			target="_blank">{source}</a
		>
	</p>
</div>

<style>
	.quote-block {
		margin-bottom: 2rem;
		color: #fff;
	}

	blockquote {
		margin: 1.3rem 0;
		color: #c5c3c3;
		font-style: italic;
	}

	.position {
		display: block;
		font-size: 0.9rem;
		font-style: italic;
		margin-top: 0.625rem;
		color: #f1f1c7;
	}

	.source a {
		color: #ffd700;
		text-decoration: none;
		font-size: 1rem;
	}

	.source a:hover {
		text-decoration: underline;
	}
</style>
