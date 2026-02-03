<script>
  import { onDestroy, onMount } from 'svelte';

  // --- BREATHING LOGIC ---
  let active = $state(false);
  let isResetting = $state(false); 
  let phaseIndex = $state(0);
  let timer = $state(4);
  let interval;

  const phases = [
    { label: 'Inhale', scale: 1.5, instruction: 'Breathe In' },
    { label: 'Hold',   scale: 1.5, instruction: 'Hold' },
    { label: 'Exhale', scale: 1.0, instruction: 'Breathe Out' },
    { label: 'Hold',   scale: 1.0, instruction: 'Hold' }
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
    setTimeout(() => { isResetting = false; }, 500);
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

  // --- INSTALL LOGIC ---
  let deferredPrompt = $state(null);
  let showIOSInstructions = $state(false);

  onMount(() => {
    // 1. Listen for the 'beforeinstallprompt' event (Android/Chrome)
    window.addEventListener('beforeinstallprompt', (e) => {
      e.preventDefault(); // Prevent the mini-infobar from appearing immediately
      deferredPrompt = e; // Stash the event so we can trigger it later
    });
  });

  async function handleInstall() {
    if (deferredPrompt) {
      // ANDROID/DESKTOP: Trigger the native prompt
      deferredPrompt.prompt();
      const { outcome } = await deferredPrompt.userChoice;
      if (outcome === 'accepted') {
        deferredPrompt = null;
      }
    } else {
      // IOS/SAFARI: Show the instruction modal
      document.getElementById('install_modal').showModal();
    }
  }
</script>

<main class="relative flex h-[100dvh] w-screen flex-col items-center justify-center bg-base-200 text-base-content overflow-hidden touch-none">
  
  <header class="absolute top-8 z-20">
    <h1 class="text-3xl font-bold tracking-widest uppercase opacity-10">Box Breathe</h1>
  </header>

  <div class="relative flex items-center justify-center w-full">
    
    <div 
      class="absolute rounded-[20%] bg-primary/30 blur-3xl transition-all ease-linear animate-pulse"
      class:duration-4000={active}
      class:duration-500={!active}
      style="width: {active ? currentPhase.scale * 60 : 55}vmin; height: {active ? currentPhase.scale * 60 : 55}vmin;"
    ></div>

    <div 
      class="z-10 flex flex-col items-center justify-center rounded-[20%] border-8 border-primary bg-base-200 text-primary shadow-sm transition-all ease-linear"
      class:duration-4000={active} 
      class:duration-500={!active}
      style="width: {active ? currentPhase.scale * 50 : 50}vmin; height: {active ? currentPhase.scale * 50 : 50}vmin;"
    >
      <div class="flex flex-col items-center justify-center text-center transition-opacity duration-1000">
        {#if active}
          <span class="text-xl uppercase tracking-widest text-base-content opacity-30 mb-2">{currentPhase.instruction}</span>
          <span class="font-mono font-bold text-base-content opacity-20" style="font-size: 15vmin;">{timer}</span>
        {:else}
          <span class="text-2xl font-medium text-base-content/70">Focus Here</span>
        {/if}
      </div>
    </div>
  </div>

  <div class="absolute bottom-8 flex w-full flex-col items-center gap-4 px-4 z-20 pb-[env(safe-area-inset-bottom)]">
    
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

    <a 
      href="https://www.buymeacoffee.com/phill_shields" 
      target="_blank" 
      rel="noreferrer" 
      class="btn btn-ghost btn-xs gap-2 text-xs opacity-40 hover:opacity-100"
    >
      Support the dev
    </a>

    <button 
      onclick={handleInstall}
      class="btn btn-ghost btn-xs gap-2 text-xs opacity-40 hover:opacity-100"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
      </svg>
      Install App
    </button>

  </div>

  <dialog id="install_modal" class="modal modal-bottom sm:modal-middle">
    <div class="modal-box text-center">
      <h3 class="font-bold text-lg">Install Box Breathe</h3>
      <p class="py-4">To install this app on your device:</p>
      
      <div class="flex flex-col gap-4 text-left bg-base-200 p-4 rounded-lg">
        <div class="flex items-center gap-3">
          <span class="badge badge-primary badge-outline">1</span>
          <span>Tap the <strong>Share</strong> icon in your browser bar.</span>
        </div>
        <div class="flex items-center gap-3">
          <span class="badge badge-primary badge-outline">2</span>
          <span>Scroll down and select <strong>"Add to Home Screen"</strong>.</span>
        </div>
      </div>

      <div class="modal-action justify-center">
        <form method="dialog">
          <button class="btn btn-wide">Got it</button>
        </form>
      </div>
    </div>
    <form method="dialog" class="modal-backdrop">
      <button>close</button>
    </form>
  </dialog>

</main>