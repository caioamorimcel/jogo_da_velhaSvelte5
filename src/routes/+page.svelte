<svelte:options runes={true} />

<script lang="ts">
	import { untrack } from 'svelte';
	import { emoji } from './emoji';

	type typeState = 'comece' | 'jogando' | 'pausado' | 'ganhou' | 'perdeu';

	let status = $state<typeState>('comece');
	let size = $state(10);
	let grid = $state(functionCreateGrid());
	let selected = $state<number[]>([]);
	let matches = $state<string[]>([]);
	let timerId = $state<number | null>(null);
	let time = $state(60);

	let derivedMaxMatches = $derived(grid.length / 2);

	function functionStartGameTimer() {
		function countdown() {
			if (status !== 'pausado') {
				time -= 1;
			}
		}
		timerId = setInterval(countdown, 1000);
	}

	function functionCreateGrid() {
		let cards = new Set<string>();
		let maxSize = size / 2;

		while (cards.size < maxSize) {
			const randomIndex = Math.floor(Math.random() * emoji.length);
			cards.add(emoji[randomIndex]);
		}

		return functionShuffle([...cards, ...cards]);
	}

	function functionShuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5);
	}

	function functionSelectedCard(cardIndex: number) {
		selected = selected.concat(cardIndex);
	}

	function functionTimeout() {
		selected = [];
	}

	function functionMatchCards() {
		const [first, second] = selected;

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first]);
		}

		setTimeout(functionTimeout, 300);
	}

	function functionResetGame() {
		if (timerId) {
			clearInterval(timerId);
		}
		grid = functionCreateGrid();
		selected = [];
		matches = [];
		timerId = null;
		time = 60;
	}

	function functionGameWon() {
		status = 'ganhou';
		functionResetGame();
	}

	function functionGameLost() {
		status = 'perdeu';
		functionResetGame();
	}

	function functionPauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (status) {
				case 'jogando':
					status = 'pausado';
					break;
				case 'pausado':
					status = 'jogando';
					break;
			}
		}
	}

	$effect(() => {
		if (status === 'jogando') {
			if (!timerId) {
				untrack(() => functionStartGameTimer());
			}
		}
	});

	$effect(() => {
		if (selected.length === 2) {
			untrack(() => functionMatchCards());
		}
	});

	$effect(() => {
		if (derivedMaxMatches === matches.length) {
			functionGameWon();
		}
	});

	$effect(() => {
		if (time === 0) {
			functionGameLost();
		}
	});

	$effect(() => {
		console.log({ status, selected, matches });
	});
</script>

<svelte:window on:keydown={functionPauseGame} />

{#if status === 'pausado'}
	<h1>Jogo pausado</h1>
{/if}

{#if status === 'comece'}
	<h1>Marcando jogo</h1>
	<button onclick={() => (status = 'jogando')}>Jogar</button>
{/if}

{#if status === 'jogando'}
	<h1 class="timer" class:pulse={time <= 10}>
		{time}
	</h1>

	<div class="matches">
		{#each matches as card, i (`q${i}`)}
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
				onclick={() => functionSelectedCard(cardIndex)}
			>
				<div class="back" class:match>{card}</div>
			</button>
		{/each}
	</div>
{/if}
{#if status === 'perdeu'}
	<h1>Você perdeu🤣</h1>
	<button onclick={() => (status = 'jogando')}>Tente novamente</button>
{/if}

{#if status === 'ganhou'}
	<h1>Você ganhou 🤩</h1>
	<button onclick={() => (status = 'jogando')}>Jogue novamente</button>
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
