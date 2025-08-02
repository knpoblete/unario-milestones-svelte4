<script>
	// shadow, plain, line, bubble
	//import wordmark from "$svg/wordmark-shadow.svg";
	import CarouselSwitcherCenter from "$components/CarouselSwitcherCenter.svelte";
	import CarouselElement from "$components/CarouselElement.svelte";

	let items = $state([
    {
		id: 1,
		color: 'yellow',
		year: 1991
    },
    {
		id: 2,
		color: 'red',
		year: 1992
    },
    {
		id: 3,
      	color: 'blue',
		year: 1993
    },
	{
		id: 4,
      	color: 'orange',
		year: 1994
    },
	{
		id: 5,
      	color: 'violet',
		year: 1995
    },
	{
		id: 6,
      	color: 'brown',
		year: 1996
    },
	] )

	items.reverse()
	items.push(items.shift())

	function reorderItems() {
		items.unshift(items.pop());
		items = [...items];
		console.log(items);
	}

</script>


<div id="switcher">
	
	<div class=element-container>
		{#each items.slice(-3) as item, i (i)}
			
			<CarouselElement color={item.color} isActive={i==1} title={item.year} inactiveLeft={i==0} inactiveRight={i==2}/>
			<!-- <span>{item.year}</span> -->
		{/each}
	</div>
	
	<CarouselSwitcherCenter on:click={reorderItems}/>
	
    
</div>


<style>
	#switcher {
		height: 20rem;
		/* width: auto; */
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		padding-top: 10em;
	}

	.element-container {
		display: flex;
		align-items: center;
		justify-content: center;
	}


	
</style>