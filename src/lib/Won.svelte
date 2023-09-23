<script>
  export let state
</script>

{#if state === 'won'}
  <h1>You Won! 🎉</h1>
  <button on:click={() => (state = 'playing')}>Play Again</button>
{/if}
