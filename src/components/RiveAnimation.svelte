<script>
	import { onMount } from "svelte";
	import * as rive from "@rive-app/canvas";

	let isJumpingInput = $state();
	let canvas = $state();

	onMount(() => {
		if (!canvas) return;

		// Create the Rive instance
		const riveInstance = new rive.Rive({
			src: "/src/animations/playfulOnboarding.riv",
			canvas,
			autoplay: true,
			stateMachines: "blob",
			onLoad: () => {
				riveInstance.resizeDrawingSurfaceToCanvas();
				const inputs = riveInstance.stateMachineInputs("blob");
				// Reference the inputs we defined in Rive
				isJumpingInput = inputs.find((i) => i.name === "isJumping");
			},
		});

		// Cleanup the Rive instance when the component unmounts
		return () => {
			riveInstance.cleanup();
		};
	});
</script>

<div class="main-container">
	<canvas bind:this={canvas} width="500" height="500"></canvas>
	<div class="headline">
		<p class="title">You're all set!</p>
		<p class="headline-text">We're excited to have you aboard</p>
	</div>
	<button
		class="cta"
		onmouseenter={() => {
			if (isJumpingInput) isJumpingInput.value = true;
		}}
		onmouseleave={() => {
			if (isJumpingInput) isJumpingInput.value = false;
		}}
	>
		<p>Hover me</p>
	</button>
</div>
