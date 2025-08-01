<script>
	import { getContext } from "svelte";
	import Footer from "$components/Footer.svelte";
	import CarouselSwitcher from "$components/CarouselSwitcher.svelte";

	// const copy = getContext("copy");
	// const data = getContext("data");
</script>

<svelte:boundary onerror={(e) => console.error(e)}>
	<!-- <Footer recirc={true} /> -->
	<CarouselSwitcher/>
</svelte:boundary>
