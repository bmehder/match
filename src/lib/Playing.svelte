<script>
  export let state
  export let time
  export let matches
  export let selected
  export let grid
  export let selectCard
</script>

{#if state === 'playing'}
  <h1 class="timer" class:pulse={time <= 10}>{time}</h1>

  <div class="cards">
    {#each grid as card, cardIndex}
      {@const isSelected = selected.includes(cardIndex)}
      {@const isMatched = matches.includes(card)}
      <button
        class="card"
        class:selected={isSelected}
        class:flip={isSelected || isMatched}
        disabled={isSelected || isMatched}
        on:click={() => selectCard(cardIndex)}
      >
        <div class="back" class:match={isMatched}>{card}</div>
      </button>
    {/each}
  </div>

  <div class="matches">
    {#each matches as card}
      <div>{card}</div>
    {/each}
  </div>
{/if}

<style>
  .cards {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 0.4rem;
  }

  .card {
    width: 140px;
    height: 140px;
    background-color: var(--bg-2);
    font-size: 4rem;
    transition: rotate 0.3s ease-out;
    transform-style: preserve-3d;

    &.flip {
      rotate: y 180deg;
      pointer-events: none;
    }

    & .back {
      position: absolute;
      inset: 0;
      display: grid;
      place-content: center;
      backface-visibility: hidden;
      rotate: y 180deg;
    }

    &.selected {
      border: 4px solid var(--border);
    }

    & .match {
      opacity: 0.4;
      transition: opacity 0.3s ease-out;
    }
  }

  .matches {
    display: flex;
    gap: 1rem;
    margin-block: 2rem;
    font-size: 3rem;
  }

  .timer {
    transition: color 0.3s ease;
  }

  .pulse {
    color: var(--pulse);
    animation: pulse 1s infinite ease-in-out;
  }

  @keyframes pulse {
    to {
      scale: 1.4;
    }
  }
</style>
