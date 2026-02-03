<script>
  import { onDestroy, onMount } from "svelte";
  import FooterButtons from "./lib/FooterButtons.svelte";

  // --- BREATHING LOGIC ---
  let active = $state(false);
  let isResetting = $state(false);
  let phaseIndex = $state(0);
  let timer = $state(4);
  let interval;

  const phases = [
    { label: "Inhale", scale: 1.5, instruction: "Breathe In" },
    { label: "Hold", scale: 1.5, instruction: "Hold" },
    { label: "Exhale", scale: 1.0, instruction: "Breathe Out" },
    { label: "Hold", scale: 1.0, instruction: "Hold" },
  ];

  let currentPhase = $derived(phases[phaseIndex]);

  function toggleSession() {
    if (isResetting) return;
    if (active) {
      stopSession();
    } else {
      startSession();
    }
  }

  function startSession() {
    active = true;
    startTimer();
  }

  function stopSession() {
    active = false;
    isResetting = true;
    stopTimer();
    setTimeout(() => {
      isResetting = false;
    }, 500);
  }

  function startTimer() {
    phaseIndex = 0;
    timer = 4;
    interval = setInterval(() => {
      timer -= 1;
      if (timer <= 0) {
        timer = 4;
        phaseIndex = (phaseIndex + 1) % 4;
      }
    }, 1000);
  }

  function stopTimer() {
    clearInterval(interval);
    phaseIndex = 0;
    timer = 4;
  }

  onDestroy(() => {
    if (interval) clearInterval(interval);
  });

</script>

<main
  class="relative flex h-[100dvh] w-screen flex-col items-center justify-center bg-base-200 text-base-content overflow-hidden touch-none"
>
  <header class="absolute top-8 z-20">
    <h1 class="text-3xl font-bold tracking-widest uppercase opacity-10">
      Breathe
    </h1>
  </header>

  <div class="relative flex items-center justify-center w-full">
    <div
      class="absolute rounded-[20%] bg-primary/30 blur-3xl transition-all ease-linear animate-pulse"
      class:duration-4000={active}
      class:duration-500={!active}
      style="width: {active ? currentPhase.scale * 60 : 55}vmin; height: {active
        ? currentPhase.scale * 60
        : 55}vmin;"
    ></div>

    <div
      class="z-10 flex flex-col items-center justify-center rounded-[20%] border-8 border-primary bg-base-200 text-primary shadow-sm transition-all ease-linear"
      class:duration-4000={active}
      class:duration-500={!active}
      style="width: {active ? currentPhase.scale * 50 : 50}vmin; height: {active
        ? currentPhase.scale * 50
        : 50}vmin;"
    >
      <div
        class="flex flex-col items-center justify-center text-center transition-opacity duration-1000"
      >
        {#if active}
          <span
            class="text-xl uppercase tracking-widest text-base-content opacity-30 mb-2"
            >{currentPhase.instruction}</span
          >
          <span
            class="font-mono font-bold text-base-content opacity-20"
            style="font-size: 15vmin;">{timer}</span
          >
        {:else}
          <span class="text-2xl font-medium text-base-content/70"
            >Focus Here</span
          >
        {/if}
      </div>
    </div>
  </div>

  <div
    class="absolute bottom-8 flex w-full flex-col items-center gap-4 px-4 z-20 pb-[env(safe-area-inset-bottom)]"
  >
    <button
      onclick={toggleSession}
      disabled={isResetting}
      class="btn btn-lg min-w-[200px] rounded-full shadow-lg transition-all active:scale-95 active:opacity-70 border-2 disabled:opacity-50 disabled:cursor-not-allowed mb-2"
      class:btn-primary={!active && !isResetting}
      class:btn-soft={active || isResetting}
      class:btn-error={active || isResetting}
    >
      {#if isResetting}
        Resetting...
      {:else if active}
        Stop Session
      {:else}
        Start Breathing
      {/if}
    </button>

    <FooterButtons />
  </div>
</main>
