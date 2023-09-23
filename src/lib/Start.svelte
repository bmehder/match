<script>
  export let state
</script>

{#if state === 'start'}
  <h1>Matching Game</h1>
  <button on:click={() => (state = 'playing')}>Play</button>
{/if}
