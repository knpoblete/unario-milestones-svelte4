<script>
	import { onMount } from "svelte";
	// import * as rive from "@rive-app/canvas";
    import * as rive from "@rive-app/webgl2";

	let isJumpingInput = $state();
	let canvas = $state();

	onMount(() => {
		if (!canvas) return;

		// Create the Rive instance
		const riveInstance = new rive.Rive({
			src: "/animations/unario_interactive_full_animation_root.riv",
			canvas,
			autoplay: true,
			stateMachines: "UNARIO Interactive",
            
			onLoad: () => {
				riveInstance.resizeDrawingSurfaceToCanvas();
				const inputs = riveInstance.stateMachineInputs("UNARIO Interactive");
				// Reference the inputs we defined in Rive
				isJumpingInput = inputs.find((i) => i.name === "isRotating");
			},
		});

        

		// Cleanup the Rive instance when the component unmounts
		return () => {
			riveInstance.cleanup();
		};
	});
</script>

<div class="main-container">
	<canvas bind:this={canvas} width="1440" height="900"></canvas>
	<div class="headline">
	</div>
	<!-- <button
		class="cta"
		onclick={() => {
            console.log(isJumpingInput.value)
			if (isJumpingInput) isJumpingInput.value = true;
		}}
		onmouseleave={() => {
			if (isJumpingInput) isJumpingInput.value = false;
		}}
	>
		<p>Click me</p>
	</button> -->
</div>
