<script>
  import emojis from './emoji'
  import { shuffle } from '$lib/utils'

  const startGameTimer = () => {
    const countdown = () => state !== 'paused' && (time -= 1)
    timerId = setInterval(countdown, 1000)
  }

  const createGrid = () => {
    let cards = new Set()
    let maxSize = size / 2

    while (cards.size < maxSize) {
      const randomIndex = Math.floor(Math.random() * emojis.length)
      cards.add(emojis[randomIndex])
    }

    return shuffle([...cards, ...cards])
  }

  const selectCard = idx => (selected = [...selected, idx])

  const matchCards = () => {
    const [first, second] = selected

    grid[first] === grid[second] && (matches = [...matches, grid[first]])

    setTimeout(() => (selected = []), 300)
  }

  const gameWon = () => {
    state = 'won'
    resetGame()
  }

  const gameLost = () => {
    state = 'lost'
    resetGame()
  }

  const pauseGame = evt => {
    if (evt.key !== 'Escape') return
    if (state === 'playing') return (state = 'paused')
    if (state === 'paused') return (state = 'playing')
  }

  const resetGame = () => {
    timerId && clearInterval(timerId)
    grid = createGrid()
    maxMatches = grid.length / 2
    selected = []
    matches = []
    timerId = null
    time = 60
  }

  let state = 'start'
  let size = 20
  let grid = createGrid()
  let maxMatches = grid.length / 2
  let selected = []
  let matches = []
  let timerId = null
  let time = 60

  // Reactively manage state
  $: selected.length === 2 && matchCards()

  $: maxMatches === matches.length && gameWon()

  $: time === 0 && gameLost()

  $: state === 'playing' && !timerId && startGameTimer()
</script>

<svelte:window on:keydown={pauseGame} />

{#if state === 'paused'}
  <h1>Game paused</h1>
{/if}

{#if state === 'start'}
  <h1>Matching Game</h1>
  <button on:click={() => (state = 'playing')}>Play</button>
{/if}

{#if state === 'playing'}
  <h1 class="timer" class:pulse={time <= 10}>{time}</h1>
  <div class="matches">
    {#each matches as card}
      <div>{card}</div>
    {/each}
  </div>

  <div class="cards">
    {#each grid as card, cardIndex}
      {@const isSelected = selected.includes(cardIndex)}
      {@const isSelectedOrMatched =
        selected.includes(cardIndex) || matches.includes(card)}
      {@const match = matches.includes(card)}
      <button
        class="card"
        class:selected={isSelected}
        class:flip={isSelectedOrMatched}
        disabled={isSelectedOrMatched}
        on:click={() => selectCard(cardIndex)}
      >
        <div class="back" class:match>{card}</div>
      </button>
    {/each}
  </div>
{/if}

{#if state === 'lost'}
  <h1>You Lost! 💩</h1>
  <button on:click={() => (state = 'playing')}>Play Again</button>
{/if}

{#if state === 'won'}
  <h1>You Won! 🎉</h1>
  <button on:click={() => (state = 'playing')}>Play Again</button>
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
    animation: pulse 1s infinite ease;
  }

  @keyframes pulse {
    to {
      scale: 1.4;
    }
  }
</style>
