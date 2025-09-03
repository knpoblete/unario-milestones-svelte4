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
  const HIDE_MS = 1300;
  let pending = $state(0);
  let draining = false;
  const sleep = ms => new Promise(r => setTimeout(r, ms));

  // Content
  let slides = $state([
    { year: "MANILA 2025", text: "It's a milestone in the making! <br> Capture the insights, connections, and moments that matter. Share your experience on LinkedIn using the hashtag <strong>#UNARIO2025.</strong>", photo: "/photos/01_Manila 2025@4x.png" },
    { year: "NEW YORK 2003", text: "First Formal Meeting: Defining the role of the Ombuds", photo: "/photos/02_New York 2003@4x.png" },
    { year: "2006 MEETING", text: "Opening Doors: Expanding Membership Beyond the UN", photo: "/photos/03_2006 Meeting@4x.png" },
    { year: "2007 MEETING", text: "Adopting a Shared Mission: Ombuds from diverse organizations dedicated to international cooperation", photo: "/photos/04_2007 Meeting@4x.png" },
    { year: "ROME 2011", text: "Solidifying UNARIO's membership and identity", photo: "/photos/05_Rome 2011@4x.png" },
    { year: "MANILA 2015", text: "Recognizing the need for a network of ombuds and mediators", photo: "/photos/06_Manila 2015@4x.png"  },
    { year: "BANGKOK 2019", text: "Promoting dignity, civility, and mental health", photo: "/photos/07_Bangkok 2019@4x.png"  },
    { year: "GENEVA 2022", text: "Strengthening Independence and Professional Standards", photo: "/photos/08_Geneva 2022@4x.png"  },
    { year: "MONTEVIDEO 2023", text: "Driving systemic change and addressing mental health", photo: "/photos/09_Montevideo 2023@4x.png",  },
    { year: "WASHINGTON, D.C. 2024", text: "Shaping the future by unlocking the potential of AI and technology", photo: "/photos/10_Washington 2024@4x.png",  },
  ]);

  let idx = $state(0);
  const idx_prev = $derived((idx - 1 + slides.length) % slides.length);
  const idx_next = $derived((idx + 1) % slides.length);

  // Rive
  const STATE_MACHINE = "UNARIO Interactive";
  const NEXT_TRIGGER_NAME = "Next";
  let nextTriggerInput = null;

  function resizeCanvasToDisplaySize() {
    if (!canvas || !riveInstance) return;
    const dpr = Math.min(window.devicePixelRatio || 1, 2);
    const { clientWidth, clientHeight } = canvas;
    canvas.width  = Math.floor(clientWidth  * dpr);
    canvas.height = Math.floor(clientHeight * dpr);
    riveInstance.resizeDrawingSurfaceToCanvas();
  }

  // ======== 🔊 SOUND: low-latency Web Audio with HTMLAudio fallback ========
  const CLICK_URL = "/sounds/click.wav"; // put file in static/sounds/click.mp3

  let audioCtx = null;      // AudioContext | null
  let clickBuffer = null;   // AudioBuffer | null
  let htmlAudio = null;     // HTMLAudioElement fallback

  async function ensureDecodedBuffer() {
    try {
      if (!audioCtx) {
        // Don't auto-start/suspend; resume happens on first user gesture
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      }
      if (!clickBuffer) {
        const res = await fetch(CLICK_URL, { cache: "force-cache" });
        const arr = await res.arrayBuffer();
        clickBuffer = await audioCtx.decodeAudioData(arr);
      }
      return true;
    } catch (e) {
      // Fallback to HTMLAudio if decoding/context fails
      if (!htmlAudio) {
        htmlAudio = new Audio(CLICK_URL);
        htmlAudio.preload = "auto";
      }
      return false;
    }
  }

  async function playClick() {
    // Ensure buffer ready (may already be decoded)
    const ok = await ensureDecodedBuffer();

    // On iOS/Safari, context may need a gesture to resume
    if (audioCtx && audioCtx.state === "suspended") {
      try { await audioCtx.resume(); } catch {}
    }

    if (ok && audioCtx && clickBuffer) {
      // Create short-lived source for zero latency
      const src = audioCtx.createBufferSource();
      src.buffer = clickBuffer;
      src.connect(audioCtx.destination);
      try { src.start(0); } catch {}
    } else if (htmlAudio) {
      try {
        // Reuse element, rewind for “machine-gun” clicking
        htmlAudio.currentTime = 0;
        await htmlAudio.play();
      } catch {}
    }
  }
  // ========================================================================

  async function onCanvasClick() {
    // 🔊 play sound immediately on click
    playClick();

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
      idx = (idx + 1) % slides.length;
      sleep(50);
    }

    // fade back in once with the final text
    isHidden = false;
    await sleep(50);

    draining = false;
  }

  onMount(() => {
    // (Optional) warm up decode in the background so first click is snappy
    // No sound will play until a user gesture due to browser audio policies.
    ensureDecodedBuffer();

    riveInstance = new rive.Rive({
      src: "/animations/[approach_2].riv",
      canvas,
      autoplay: true,
      stateMachines: STATE_MACHINE,
      layout: new rive.Layout({
        fit: rive.Fit.Contain,
        alignment: rive.Alignment.Center,
      }),
      autoBind: true,
      onLoad: () => {
        console.log("Rive loaded");

        // Grab inputs and cache the 'Next' trigger if you want it
        const inputs = riveInstance.stateMachineInputs(STATE_MACHINE) ?? [];
        inputs.forEach((input) => {
          if (input.name === NEXT_TRIGGER_NAME) nextTriggerInput = input;
        });

        resizeCanvasToDisplaySize();
      },
      onStateChange: () => {
        // When Rive reports the button press, advance & play sound
        const inputs = riveInstance.stateMachineInputs(STATE_MACHINE) ?? [];
        inputs.forEach((input) => {
          if (input.name == "isDown" && input.value == true) {
            // 🔊 sound on Rive-press, too
            playClick();
            onCanvasClick();
          }
        });
      },
    });

    // canvas.addEventListener("pointerdown", onCanvasClick, { passive: true });

    ro = new ResizeObserver(resizeCanvasToDisplaySize);
    ro.observe(canvas);

    return () => {
      ro?.disconnect();
      canvas?.removeEventListener("pointerdown", onCanvasClick);
      riveInstance?.cleanup();
    };
  });

  onDestroy(() => {
    clearTimeout(hideTO);
    try { audioCtx?.close(); } catch {}
  });

  onDestroy(() => {
    clearTimeout(hideTO);
  });
</script>


<div class="scrollport">
  <div class="stage" role="img" aria-label="1920×1080 stage">
    <!-- Canvas (Rive draws here) -->
    <canvas
      bind:this={canvas}
      class="rive"
      aria-label="Rive canvas (click to change text)"
    ></canvas>

    <!-- SVG overlay (same box as the canvas) -->
    <div class="overlay-svg" aria-hidden="true">
      {@html background_svg}
    </div>

    <!-- Your text overlays -->
    {#if slides.length}
      <div class="overlay-text-center" class:hidden={isHidden} aria-live="polite" aria-hidden={isHidden}>
        <img src={slides[idx].photo} />
        <h1>{slides[idx].year}</h1>
        <p>{@html slides[idx].text}</p>
      </div>

      <div class="overlay-text-left" class:hidden={isHidden} aria-live="polite" aria-hidden={isHidden}>
        <img src={slides[idx_prev].photo} />
        <h1>{slides[idx_prev].year}</h1>
        <p>{@html slides[idx_prev].text}</p>
        
      </div>

      <div class="overlay-text-right" class:hidden={isHidden} aria-live="polite" aria-hidden={isHidden}>
        <!-- future right-side content -->
      </div>
    {/if}
  </div>
</div>

<style>
  /* Make the viewport fill the window */
  :global(html, body) { margin: 0; height: 100%; }
  * { box-sizing: border-box; }

  /* Scrolls when the window is smaller than 1920×1080 */
  .scrollport {
    width: 100vw;
    height: 100vh;
    overflow: auto;            /* shows scrollbars only when needed */
    overscroll-behavior: contain;
  }

  /* Fixed-size stage that both canvas and SVG align to */
  .stage {
    position: relative;
    width: 1920px;
    height: 1080px;
    margin: 0;                 /* keep (0,0) at top-left for coordinates */
    
  }

  /* Stack the drawing layers perfectly */
  .rive,
  .overlay-svg {
    position: absolute;
    inset: 0;                  /* top/right/bottom/left: 0 */
    width: 100%;
    height: 100%;
    display: block;
  }

  .rive {
    z-index: 1;
    cursor: pointer;           /* hint that canvas is interactive */
  }

  .overlay-svg {
    z-index: 2;
    pointer-events: none;      /* let clicks go through to the canvas */
  }

  /* Ensure injected SVG fills the overlay box */
  :global(.overlay-svg svg) {
    width: 100%;
    height: 100%;
    display: block;
  }

  /* Optional: center the stage when there’s extra room */
  @media (min-width: 1920px) and (min-height: 1080px) {
    .scrollport {
      display: grid;
      place-items: start center;
    }
  }

  /* ===== Overlay text styles (keep from your original, adjusted to sit on .stage) ===== */

  .overlay-text-center,
  .overlay-text-left,
  .overlay-text-right {
    position: absolute;
    z-index: 3;
    pointer-events: none; /* pass clicks through to canvas */
    color: #0099D8;
    font-family: "Ideal Sans", system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    gap: 0.5rem;
    opacity: 1;
    transition: opacity 0.2s ease;
  }

  .overlay-text-center {
    max-width: 480px;
    top: 15%;
    left: 62%;
    right: auto;
    bottom: auto;
    display: flex;
    flex-direction: column;
    text-align: left;
    padding: 2rem;
  }
  .overlay-text-center.hidden { opacity: 0; }
  .overlay-text-center img { display: block; max-width: 100%; height: auto; margin: 0; }
  .overlay-text-center h1 {
    font-weight: 600;
    font-size: clamp(1.5rem, 6vw, 2.5rem);
    margin: 0;
    line-height: 0.8;
  }
  .overlay-text-center p {
    font-weight: 400;
    color: #989898;
    font-size: clamp(1.25rem, 2.5vw, 1.5rem);
    margin: 0;
    line-height: 1.2;
    opacity: 0.9;
  }

  .overlay-text-left {
    display: inline-block;
    transform: rotate(-45deg);
    max-width: 450px;
    top: 44%;
    left: 26%;
    right: auto;
    bottom: auto;
    text-align: left;
    padding: 2rem;
    transition: opacity 0.1s ease; /* faster fade if you like */
    -webkit-mask-repeat: no-repeat;
    -webkit-mask-size: 100% 100%;
    mask-image: linear-gradient(to left, black 1%, transparent 40%);
    mask-repeat: no-repeat;
    mask-size: 100% 100%;
  }
  .overlay-text-left.hidden { opacity: 0; }
  .overlay-text-left img { display: block; max-width: 100%; height: auto; margin: 0; }
  .overlay-text-left h1 {
    font-weight: 600;
    font-size: clamp(1.5rem, 6vw, 2.5rem);
    margin: 0;
    line-height: 1.1;
  }
  .overlay-text-left p {
    font-weight: 400;
    color: #989898;
    font-size: clamp(1.25rem, 2.5vw, 1.5rem);
    margin: 0;
    line-height: 1.3;
    opacity: 0.9;
  }

  .overlay-text-right {
    display: inline-block;
    transform: rotate(45deg);
    max-width: 450px;
    top: 45%;
    left: 98%;
    right: auto;
    bottom: auto;
    text-align: left;
    padding: 2rem;
    transition: opacity 0.1s ease;
    -webkit-mask-repeat: no-repeat;
    -webkit-mask-size: 100% 100%;
    mask-image: linear-gradient(to right, black 1%, transparent 40%);
    mask-repeat: no-repeat;
    mask-size: 100% 100%;
  }
  .overlay-text-right.hidden { opacity: 0; }
  .overlay-text-right h1 {
    font-weight: 600;
    font-size: clamp(1.5rem, 6vw, 2.5rem);
    margin: 0;
  }
  .overlay-text-right p {
    font-weight: 400;
    color: #989898;
    font-size: clamp(1.25rem, 2.5vw, 1.2rem);
    margin: 0;
    opacity: 0.9;
  }
</style>
