<script>
  export let state

  function pauseGame(evt) {
    if (evt.key !== 'Escape') return
    return (state = state === 'playing' ? 'paused' : 'playing')
  }
</script>

<svelte:window on:keydown={pauseGame} />

{#if state === 'paused'}
  <h1>Game paused</h1>
{/if}
