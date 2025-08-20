<script>
  import { onMount, onDestroy } from "svelte";
  import * as rive from "@rive-app/webgl2";
  import background_svg from "$svg/background.svg";

  let canvas = $state();
  let riveInstance;
  let ro;

  // Visibility + timing
  let isHidden = $state(false);
  let hideTO;
  let animating = $state(false);
  const HIDE_MS = 1400; // 0.2s (matches CSS transition)
  let pending = $state(0);
let draining = false;
const sleep = ms => new Promise(r => setTimeout(r, ms));


  // Content
  let slides = $state([
    { year: "2019", text: "text 1" },
    { year: "2020", text: "text 2" },
    { year: "2021", text: "text 3" },
    { year: "2022", text: "text 4" },
    { year: "2023", text: "text 5" },
    { year: "2024", text: "text 6" },
    { year: "last", text: "text" },
  ]);

  let idx = $state(0);

  // Rive
  const STATE_MACHINE = "UNARIO Interactive";
  const NEXT_TRIGGER_NAME = "Next";
  let nextTriggerInput = null;

  function nextSlide() {
    idx = (idx + 1) % slides.length;
  }

  function resizeCanvasToDisplaySize() {
    if (!canvas || !riveInstance) return;
    const dpr = Math.min(window.devicePixelRatio || 1, 2);
    const { clientWidth, clientHeight } = canvas;
    canvas.width  = Math.floor(clientWidth  * dpr);
    canvas.height = Math.floor(clientHeight * dpr);
    riveInstance.resizeDrawingSurfaceToCanvas();
  }


function onCanvasClick() {
  // record this click and kick Rive right now
  pending += 1;
  nextTriggerInput?.fire?.();

  // start the drain loop if not already running
  if (!draining) drain();
}

async function drain() {
  draining = true;

  // fade out once, then apply all queued steps while hidden
  isHidden = true;
  await sleep(HIDE_MS);

  while (pending > 0) {
    pending -= 1;
    idx = (idx + 1) % slides.length; // apply exactly one step per click
    // tiny gap so multiple triggers don't collapse into one visual frame
    await sleep(50);
  }

  // fade back in once with the final text
  isHidden = false;
  // give the fade-in a moment before accepting a new drain
  await sleep(50);

  draining = false;
}

  onMount(() => {
    riveInstance = new rive.Rive({
      src: "/animations/[approach_1].riv",
      canvas,
      autoplay: true,
      stateMachines: STATE_MACHINE,
      layout: new rive.Layout({
        fit: rive.Fit.COVER,
        alignment: rive.Alignment.Center,
      }),
      onLoad: () => {
        try {
          const inputs = riveInstance.stateMachineInputs(STATE_MACHINE) ?? [];
          nextTriggerInput =
            inputs.find(i => i.name === NEXT_TRIGGER_NAME && i.type === "trigger") || null;
        } catch {
          nextTriggerInput = null;
        }
        resizeCanvasToDisplaySize();
      },
    });

    canvas.addEventListener("pointerdown", onCanvasClick, { passive: true });

    ro = new ResizeObserver(resizeCanvasToDisplaySize);
    ro.observe(canvas);

    return () => {
      ro?.disconnect();
      canvas?.removeEventListener("click", onCanvasClick);
      riveInstance?.cleanup();
    };
  });

  onDestroy(() => {
    clearTimeout(hideTO);
  });
</script>

<div class="outer">
  <div class="top" aria-hidden="true">{@html background_svg}</div>

  <div class="wheel">
    <canvas
      bind:this={canvas}
      class="rive"
      fit="cover"
      aria-label="Rive canvas (click to change text)"
    ></canvas>

    {#if slides.length}
      <div
        class="overlay-text"
        class:hidden={isHidden}
        aria-live="polite"
        aria-hidden={isHidden}
      >
        <h1>{slides[idx].year}</h1>
        <p>{slides[idx].text}</p>
      </div>
    {/if}
  </div>
</div>

<style>
  .wheel {
    position: relative;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  .top {
    width: 100%;
    height: 100%;
    overflow: hidden;
    z-index: 2;
    pointer-events: none;
  }

  .rive {
    width: 100%;
    height: 100%;
    display: block;
    cursor: pointer; /* hint that canvas is interactive */
  }

  .overlay-text {
    position: absolute;
    top: 30%;
    left: 56%;
    right: auto;
    bottom: auto;
    /* transform: translate(-50%, -50%);  // use if you prefer center anchoring */

    display: flex;
    flex-direction: column;
    text-align: left;
    padding: 2rem;
    z-index: 3;
    pointer-events: none; /* pass clicks through to canvas */
    color: black;
    gap: 0.5rem;

    opacity: 1;
    transition: opacity 0.1s ease; /* match HIDE_MS */
  }

  .overlay-text.hidden { opacity: 0; }

  .overlay-text h1 {
    font-size: clamp(2rem, 6vw, 4rem);
    margin: 0;
    line-height: 1.1;
  }

  .overlay-text p {
    font-size: clamp(1rem, 2.5vw, 1.25rem);
    margin: 0;
    line-height: 1.3;
    opacity: 0.9;
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

  :global(html, body) { margin: 0; }
</style>
