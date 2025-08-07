<script>
	import "$styles/app.css";
	// import Header from "$components/Header.svelte";

	let { children } = $props();
</script>

<!-- <Header /> -->
<main id="content">
	{@render children?.()}
</main>

<style>
  :global(html, body) {
    margin: 0;
    padding: 0;
    height: 100%;
  }

  :global(body) {
    overflow: hidden; /* Optional, prevents scrollbars */
  }
</style>