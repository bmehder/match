<script>
  import '../app.css'
  import emojis from './emojis'
  import Paused from '$lib/Paused.svelte'
  import Start from '../lib/Start.svelte'
  import Won from '../lib/Won.svelte'
  import Lost from '../lib/Lost.svelte'

  const shuffle = xs => xs.sort(() => Math.random() - 0.5)

  let state = 'start'
  let size = 20
  let grid = createGrid()
  let maxMatches = grid.length / 2
  let selected = []
  let matches = []
  let timerId = null
  let time = 60

  function createGrid() {
    let cards = new Set()

    ;(function addCards(x) {
      const randomIndex = Math.floor(Math.random() * emojis.length)
      cards.add(emojis[randomIndex])

      if (x < size / 2) addCards(cards.size + 1)
    })(cards.size)

    return shuffle([...cards, ...cards])
  }

  function startGameTimer() {
    timerId = setInterval(() => state !== 'paused' && (time -= 1), 1000)
  }

  function selectCard(x) {
    selected = [...selected, x]
  }

  function matchCards() {
    const [first, second] = selected

    grid[first] === grid[second] && (matches = [...matches, grid[first]])

    setTimeout(() => (selected = []), 300)
  }

  function gameWon() {
    state = 'won'
    resetGame()
  }

  function gameLost() {
    state = 'lost'
    resetGame()
  }

  function pauseGame(evt) {
    if (evt.key !== 'Escape') return
    return (state = state === 'playing' ? 'paused' : 'playing')
  }

  function resetGame() {
    timerId && clearInterval(timerId)
    grid = createGrid()
    maxMatches = grid.length / 2
    selected = []
    matches = []
    timerId = null
    time = 60
  }

  $: selected.length === 2 && matchCards()

  $: maxMatches === matches.length && gameWon()

  $: time === 0 && gameLost()

  $: state === 'playing' && !timerId && startGameTimer()
</script>

<svelte:window on:keydown={pauseGame} />

<main>
  <Start bind:state />
  
  <Paused bind:state />
  
  <Won bind:state />

  <Lost bind:state />

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
</main>

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
