<script>
	import { onMount } from "svelte";
	import * as rive from "@rive-app/webgl2";
	import background_svg from "/animations/background.svg";

  let canvas = $state();
  let riveInstance;
  let ro; // ResizeObserver

  function resizeCanvasToDisplaySize() {
    if (!canvas || !riveInstance) return;
    const dpr = Math.min(window.devicePixelRatio || 1, 2); // cap for perf
    // CSS controls the *display* size; use that to set the pixel buffer
    const { clientWidth, clientHeight } = canvas;
    canvas.width  = Math.floor(clientWidth  * dpr);
    canvas.height = Math.floor(clientHeight * dpr);
    riveInstance.resizeDrawingSurfaceToCanvas();
  }

  onMount(() => {
    // IMPORTANT: put your .riv in /static and reference like /animations/...
    riveInstance = new rive.Rive({
      src: "/animations/[approach_1].riv",
      canvas,
      autoplay: true,
      stateMachines: "UNARIO Interactive",
      layout: new rive.Layout({
        fit: rive.Fit.COVER,              // fills the canvas; can crop
        alignment: rive.Alignment.Center, // center the artboard
      }),
      onLoad: () => {
        resizeCanvasToDisplaySize();
        // Access inputs if you need them:
        // const inputs = riveInstance.stateMachineInputs("UNARIO Interactive");
        // const isRotating = inputs.find((i) => i.name === "isRotating");
      },
    });

    ro = new ResizeObserver(resizeCanvasToDisplaySize);
    ro.observe(canvas);

    return () => {
      ro?.disconnect();
      riveInstance?.cleanup();
    };
  });
</script>


<div class="outer">
  <div class="top" aria-hidden="true">{@html background_svg}</div>
  <div class="wheel">
  	<canvas bind:this={canvas} class="rive"></canvas>
	</div>
</div>

<style>
  /* Full-bleed section that spans the page width */
  .wheel {
    width: 100vw;     /* full width of the viewport */
    height: 60vh;     /* pick your height: 50–80vh is a nice hero */
    overflow: hidden; /* hide cropped edges when using COVER */
  }

  .top {
    width: 100vw;     /* full width of the viewport */
    height: 60vh;     /* pick your height: 50–80vh is a nice hero */
    overflow: hidden; /* hide cropped edges when using COVER */
  }

  /* Make the canvas fill its container; JS sets the pixel buffer */
  .rive {
    width: 100%;
    height: 100%;
    display: block;
  }

  .outer {
  display: grid;
  grid-template: 1fr / 1fr;
  place-items: center;
}
.outer > * {
  grid-column: 1 / 1;
  grid-row: 1 / 1;
}
.outer .below {
  z-index: 1;
  /* background-color: aqua; */
}
.outer .top {
  z-index: 2;
  pointer-events: none;
  /* background-color: brown; */
}

  /* Optional: remove default body margin if you want true edge-to-edge */
  :global(html, body) { margin: 0; }

</style>
