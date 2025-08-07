<script>
	import { onMount } from "svelte";
	// import * as rive from "@rive-app/canvas";
    import * as rive from "@rive-app/webgl2";

	let isJumpingInput = $state();
	let canvas = $state();
    let ctx;

	onMount(() => {
		if (!canvas) return;

		// Create the Rive instance
		const riveInstance = new rive.Rive({
			src: "/animations/wheel_animation_kath.riv",
			// src: "/animations/unario_interactive_full_animation_data_bind_rotate_b3.riv",
			canvas,
			autoplay: true,
			stateMachines: "UNARIO Interactive",
            
			onLoad: () => {
				riveInstance.resizeDrawingSurfaceToCanvas();
				const inputs = riveInstance.stateMachineInputs("UNARIO Interactive");
				
				// Use your trigger input here
				// isJumpingInput = inputs.find((i) => i.name === "isHovering");
			},

		});
        

		// Cleanup the Rive instance when the component unmounts
		return () => {
			riveInstance.cleanup();
		};
	});

</script>

<div class="main-container">

	<canvas bind:this={canvas} width="100%" height="100%"></canvas>

	
</div>


<style>
  canvas {
    width: 100vw;
    height: 100vh;
    display: block;
	
  }

  /* :global(body, html) {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  :global(main) {
    height: 100vh;
  } */
</style>