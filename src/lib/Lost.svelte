<script>
  export let state
</script>

{#if state === 'lost'}
  <h1>You Lost! 💩</h1>
  <button on:click={() => (state = 'playing')}>Play Again</button>
{/if}
