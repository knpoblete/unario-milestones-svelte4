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
		margin: 7em;
    /* color: gray; */
    transform: scale(0.7);
    opacity: 0.5;
	}

  .quote.active {
    /* color: red; */
    margin: 10em;
    transform: scale(1.6);
    opacity: 1;
  }

  .quote.inactive-left {
    /* color: blue; */
    transform: rotate(-45deg)
  }

  .quote.inactive-right {
    /* color: yellow; */
    transform: rotate(45deg);
  }

  .quote > span {
    color: black;
    font-weight: bold;
    z-index: 1000;
    display:block;
  }


</style>

