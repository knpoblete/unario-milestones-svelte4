<script>
	// shadow, plain, line, bubble
	//import wordmark from "$svg/wordmark-shadow.svg";
	import wordmark from "$svg/unario-logo.svg";
</script>

<header>
	<div class="wordmark">
		<a href="https://pudding.cool" aria-label="The Pudding" target="_self"
			>{@html wordmark}</a
		>
	</div>
</header>

<style>
	.wordmark {
		max-width: 14em;
		/* margin: 0 auto; */
		padding: 32px 0;
		transform: rotate(-4deg);
		margin-bottom: 20rem;
		margin-top: auto;
		margin-left: auto;
		margin-right: auto;
	}

	.wordmark a {
		border: none;
		display: block;
		color: var(--color-fg);
	}

	.wordmark a:hover {
		background-color: transparent;
	}
</style>
