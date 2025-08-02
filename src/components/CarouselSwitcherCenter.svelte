<script>
	// shadow, plain, line, bubble
	//import wordmark from "$svg/wordmark-shadow.svg";
	import switcher from "$svg/switcher.svg";
	import { createEventDispatcher } from 'svelte';
  	const dispatch = createEventDispatcher();
</script>


<div id="switcher-button" on:click={() => dispatch('click')}>
  {@html switcher}
</div>


<style>
	#switcher-button {
		height: 15rem;
		width: 15rem;
		color: #d2d2d2;
		margin-top: -8rem;
	}

	#switcher-button:hover {
		animation: gelatine 1.5s infinite 0s;
	}
	
	
	@keyframes gelatine {
	from, to { transform: scale(1, 1); }
	25% { transform: scale(0.9, 1.1); }
	50% { transform: scale(1.1, 0.9); }
	75% { transform: scale(0.95, 1.05); }
	}


</style>