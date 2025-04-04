<!-- <svelte:options runes={true} /> -->

<script lang="ts">
	import { emoji } from './emoji';

	type State = 'comece' | 'jogando' | 'pausado' | 'ganhou' | 'perdeu';

	let state: State = 'comece';
	let size = 10;
	let grid = createGrid();
	let maxMatches = grid.length / 2;
	let selected: number[] = [];
	let matches: string[] = [];
	let timerId: number | null = null;
	let time = 60;

	function startGameTimer() {
		function countdown() {
			if (state !== 'pausado') {
				time -= 1;
			}
		}
		timerId = setInterval(countdown, 1000);
	}

	function createGrid() {
		let cards = new Set<string>();
		let maxSize = size / 2;

		while (cards.size < maxSize) {
			const randomIndex = Math.floor(Math.random() * emoji.length);
			cards.add(emoji[randomIndex]);
		}

		return shuffle([...cards, ...cards]);
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5);
	}

	function selectedCard(cardIndex: number) {
		selected = selected.concat(cardIndex);
	}

	function timeoutfunction() {
		selected = []; // Limpa a seleção sem disparar reatividade
	}

	function matchCards() {
		const [first, second] = selected;

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first]);
		}

		setTimeout(timeoutfunction, 300);
	}

	function resetGame() {
		if (timerId) {
			clearInterval(timerId);
		}
		grid = createGrid();
		maxMatches = grid.length / 2;
		selected = [];
		matches = [];
		timerId = null;
		time = 60;
	}

	function gameWon() {
		state = 'ganhou';
		resetGame();
	}

	function gameLost() {
		state = 'perdeu';
		resetGame();
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (state) {
				case 'jogando':
					state = 'pausado';
					break;
				case 'pausado':
					state = 'jogando';
					break;
			}
		}
	}

	$: if (state === 'jogando') {
		if (!timerId) {
			startGameTimer();
		}
	}

	$: {
		if (selected.length === 2) {
			matchCards();
		}
	}
	$: {
		if (maxMatches === matches.length) {
			gameWon();
		}
	}
	$: {
		if (time === 0) {
			gameLost();
		}
	}
	$: console.log({ state, selected, matches });
</script>

<svelte:window on:keydown={pauseGame} />

{#if state === 'pausado'}
	<h1>Jogo pausado</h1>
{/if}

{#if state === 'comece'}
	<h1>Marcando jogo</h1>
	<button on:click={() => (state = 'jogando')}>Jogar</button>
{/if}

{#if state === 'jogando'}
	<h1 class="timer" class:pulse={time <= 10}>
		{time}
	</h1>

	<div class="matches">
		{#each matches as card (card)}
			<div>{card}</div>
		{/each}
	</div>

	<div class="cards">
		{#each grid as card, cardIndex (cardIndex)}
			{@const isSelected = selected.includes(cardIndex)}
			{@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(card)}
			{@const match = matches.includes(card)}

			<button
				class="card"
				class:selected={isSelected}
				class:flip={isSelectedOrMatched}
				disabled={isSelectedOrMatched}
				on:click={() => selectedCard(cardIndex)}
			>
				<div class="back" class:match>{card}</div>
			</button>
		{/each}
	</div>
{/if}
{#if state === 'perdeu'}
	<h1>Você perdeu🤣</h1>
	<button on:click={() => (state = 'jogando')}>Tente novamente</button>
{/if}

{#if state === 'ganhou'}
	<h1>Você ganhou 🤩</h1>
	<button on:click={() => (state = 'jogando')}>Jogue novamente</button>
{/if}

<style>
	.cards {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 0.4rem;
	}

	.card {
		height: 140px;
		width: 140px;
		font-size: 4rem;
		background-color: var(--bg-2);
		transition: rotate 0.3s ease-out;
		transform-style: preserve-3d;

		&.selected {
			border: 4px solid var(--border);
		}

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

		& .match {
			transition: opacity 0.3s ease-out;
			opacity: 0, 4;
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
