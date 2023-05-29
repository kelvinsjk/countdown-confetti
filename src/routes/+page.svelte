<script lang="ts">
  import {Fireworks, type FireworksOptions} from 'fireworks-js';
  import { Confetti } from 'svelte-confetti';
  import {FlipDown} from './flipdown'
  import { Switch, Button } from '@svelteuidev/core';
  import { onMount } from 'svelte';
  import {fade,scale} from 'svelte/transition'
  import timer from './timer';
  
  let enabled = false;
  let options: FireworksOptions = {sound: {enabled, files: ['fireworks1.mp3', 'fireworks2.mp3']}, }
  let fireworks: Fireworks;
  let fireworksContainer: HTMLDivElement;

  let flipdown: any;
  let startConfetti = false;
  let countingDown = true;
  const endDate = new Date("June 9, 2023 17:00:00");
  //const endDate = new Date("May 29, 2023 23:44:10");
  let date = (endDate.getTime() / 1000) + 1;
  let real = false;


  onMount(()=>{
    fireworks = new Fireworks(fireworksContainer, options)
  })

  $: updateOptions(enabled)

  const clock = timer({interval:200});
  $: title = updateTitle($clock, countingDown);


  function updateTitle(date: Date, countdown: boolean): string {
    if (!countdown){
      return 'Congratulations';
    }
    const diff = Math.ceil((endDate.getTime() - date.getTime() + 1000) / (1000));
    if (diff < 60){
      return `${diff} second${diff===1 ? '' : 's'} left!`
    }
    const minutes = Math.ceil(diff/60)
    if (diff < 3600){
      return `${minutes} minute${minutes===1 ? '' : 's'} left!`
    }
    const hours = Math.ceil(minutes/60)
    if (diff < 3600*24){
      return `${hours} hour${hours===1 ? '' : 's'} left!`
    }
    const days = Math.ceil(hours/24);
    return `${days} day${days===1 ? '' : 's'} left!`
  }

  function mini() {
    fireworks.start();
    countingDown = false;
    startConfetti = true;
    window.setTimeout(()=>{
      countingDown = true;
      startConfetti = false;
      fireworks.stop();
    }, 5000)
  }

  function updateOptions(enabled: boolean){
    const files = real ? ['rocket.mp3', 'burst.mp3', 'applause.mp3'] : ['rocket.mp3', 'burst.mp3'];
    options = {sound: {enabled, files}};
    if (fireworks){
      fireworks.updateOptions(options)
    }
  }

  function setup(node:HTMLElement) {
    if (flipdown){
      node.firstChild?.remove();
      node.insertBefore(flipdown.element, node.firstChild)
    } else {
      flipdown = new FlipDown(date).start().ifEnded(()=>{
        fireworks.updateOptions({sound: {enabled, files: ['rocket.mp3', 'burst.mp3', 'applause.mp3']}});
        real = true;
        countingDown = false;
        fireworks.start();
        startConfetti = true;
      });
    }
  }
</script>

<svelte:head>
  <title>{title}</title>
  <link rel="preload" as="image" href="/tulips.png"/>
</svelte:head>

<main>
  <div 
    class="fireworks"
    bind:this={fireworksContainer}
  />
  {#if startConfetti}
  <div class="confetti">
    <Confetti x={[-5, 5]} y={[-1, 2]} amount=150 cone iterationCount=infinite delay={[750, 2000]} /> 
  </div>
  {/if}
  {#if countingDown}
  <div class="flipdown-container" transition:fade use:setup>
    <div id="flipdown" class="flipdown" />
    <Button variant='gradient' gradient={{from: 'yellow', to: 'grape', deg: 45}} ripple
      on:click={mini}
    >
      Mini Celebration
    </Button>
  </div>  
  {:else}
  <div class="flipdown-container" in:scale={{delay:500, duration:1000}}>
    <h2 class="huge">Congratulations!</h2>
    {#if real}
      <img alt="tulips" src="/tulips.png" />
    {/if}
  </div>
  {/if}
  <div class="switch">
    <Switch 
      bind:checked={enabled}
      label="🔊"
    />
  </div>
</main>

<style>
  .switch {
    position: fixed;
    top: 0;
    left: 0;
    display: flex;
    gap: 0.5rem;
    margin: 1rem;
    align-items: flex-end;
  }
  .fireworks {
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
  }
  .confetti {
    position: fixed;
    top: 40%;
    left: 0;
    height: 100vh;
    width: 100vw;
    display: flex;
    justify-content: center;
  }
  .huge {
    font-size: 2rem;
  }
  .flipdown-container {
    width: 100%;
    height: 100vh;
    height: 100dvh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 2rem;
  }
  :global(.flipdown.flipdown__theme-dark .rotor-group-heading::before) {
    color: white;
  }
  :global(body) {
    margin: 0;
    background-color: black;
    color: white;
  }
</style>