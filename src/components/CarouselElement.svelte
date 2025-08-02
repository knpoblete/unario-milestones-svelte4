<script>
  
  // let isActive = true;
  let { isActive = false, inactiveLeft = false, inactiveRight = false, color = "gray", title = "" } = $props();
  

  // console.log(isActive)
	import quote from "$svg/quote.svg?raw"; 
  
</script>

<!-- <div class="quote" class:active={isActive} class:inactive-left={inactiveLeft} class:inactive-right={inactiveRight} > -->
<div class="quote" class:active={isActive} class:inactive-left={inactiveLeft} class:inactive-right={inactiveRight} style="color: {color}">  
  {@html quote}
  <span> {title} </span>
  <!-- {isActive ? 'Active' : 'Inactive'} -->
</div>


<style> 
	.quote {
		height: 14rem;
		width: 14rem;
		margin: 20em 7em 7em 7em;
    /* color: gray; */
    transform: scale(0.7);
    opacity: 0.5;
	}

  .quote.active {
    /* color: red; */
    margin: -14em 10em 10em 10em;
    transform: scale(1.6);
    opacity: 1;
		/* animation: gelatine 1.5s 2 0s; */
	}
	
	
  @keyframes gelatine {
    from, to { transform: scale(1, 1); }
    25% { transform: scale(0.9, 1.1); }
    50% { transform: scale(1.1, 0.9); }
    75% { transform: scale(0.95, 1.05); }
  }

  .quote.inactive-left {
    /* color: blue; */
    transform: rotate(-60deg);
  }

  .quote.inactive-right {
    /* color: yellow; */
    transform: rotate(60deg);
  }

  .quote > span {
    color: black;
    font-weight: bold;
    z-index: 1000;
    display:block;
  }


</style>

