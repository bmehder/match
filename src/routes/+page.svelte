<script>
  import '../app.css'
  import emojis from './emojis'
  import Paused from '$lib/Paused.svelte'
  import Start from '../lib/Start.svelte'
  import Won from '../lib/Won.svelte'
  import Lost from '../lib/Lost.svelte'
  import Playing from '../lib/Playing.svelte'

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

  <Playing bind:state {time} {matches} {selected} {grid} {selectCard} />
</main>