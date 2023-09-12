<script>
	import { onMount } from 'svelte';

	let width = 30;
	let height = 30;
	let grid = Array.from({ length: height }, () => Array(width).fill(0));
	let interval;
	let speed = 500; //ms per generation

	onMount(() => {
		randomizeGrid();
	});

	function step() {
		const newGrid = [...Array(height)].map((_, y) =>
			[...Array(width)].map((_, x) => {
				const neighbors = [
					[-1, -1],
					[-1, 0],
					[-1, 1],
					[0, -1],
					[0, 1],
					[1, -1],
					[1, 0],
					[1, 1]
				].map(([dx, dy]) => grid[(y + dy + height) % height][(x + dx + width) % width]);

				const alive = grid[y][x] === 1;
				const sum = neighbors.reduce((a, b) => a + b, 0);

				return alive ? (sum === 2 || sum === 3 ? 1 : 0) : sum === 3 ? 1 : 0;
			})
		);
		grid = newGrid;
	}

	function toggleCell(y, x) {
		grid[y][x] = grid[y][x] === 1 ? 0 : 1;
	}

	function start() {
		if (!interval) {
			interval = setInterval(step, speed);
		}
	}

	function pause() {
		if (interval) {
			clearInterval(interval);
			interval = null;
		}
	}

	function randomizeGrid() {
		grid = Array.from({ length: height }, () =>
			Array(width)
				.fill(0)
				.map(() => (Math.random() < 0.5 ? 0 : 1))
		);
	}

	function clearGrid() {
		grid = Array.from({ length: height }, () => Array(width).fill(0));
	}

	function updateSpeed(event) {
		speed = event.target.value;
		pause();
		start();
	}
</script>

<div class="controls">
	<button on:click={start}>Start</button>
	<button on:click={pause}>Pause</button>
	<button on:click={step}>Step</button>
	<button on:click={randomizeGrid}>Randomize</button>
	<button on:click={clearGrid}>Clear</button>
	<label>
		Speed:
		<input type="range" min="50" max="1000" step="50" bind:value={speed} on:input={updateSpeed} />
	</label>
</div>

<div class="grid" style="grid-template-columns: repeat({width}, 1fr);">
	{#each grid as row, y}
		{#each row as cell, x}
			<div class="cell {cell === 1 ? 'alive' : ''}" on:click={() => toggleCell(y, x)} />
		{/each}
	{/each}
</div>

<style>
	.grid {
		display: grid;
		gap: 1px;
		border: 1px solid #eee;
	}
	.cell {
		width: 20px;
		height: 20px;
		background: #eee;
		cursor: pointer;
	}
	.cell.alive {
		background: #333;
	}
	button {
		margin: 5px;
	}
</style>
