<script lang="ts">
	import { onMount } from 'svelte';
	import { Wheel } from 'spin-wheel';
	import confetti from 'canvas-confetti';
	import Modal from './Modal.svelte';

	let wheelContainer: HTMLDivElement;
	let wheel: Wheel;
	let optionsText = 'Option 1\nOption 2\nOption 3\nOption 4\nOption 5\nOption 6';
	let eliminationMode = false;
	let winners: string[] = [];
	let showWinnerModal = false;
	let currentWinner = '';
	let showResetModal = false;
	let loading = true;

	$: items = optionsText.split('\n').filter(line => line.trim()).map(label => ({ label: label.trim() }));
	$: if (typeof window !== 'undefined' && !loading) {
		localStorage.setItem('spinwheelOptions', optionsText);
		localStorage.setItem('spinwheelWinners', JSON.stringify(winners));
		localStorage.setItem('spinwheelEliminationMode', JSON.stringify(eliminationMode));
	}

	function handleWinner() {
		confetti();
		const winner = wheel.getCurrentIndex();
		currentWinner = items[winner].label;
		showWinnerModal = true;
	}

	function closeWinnerModal() {
		if (eliminationMode) {
			winners = [...winners, currentWinner];
			optionsText = optionsText.split('\n').filter(line => line.trim() !== currentWinner).join('\n');
		}
		showWinnerModal = false;
	}

	onMount(() => {
		if (typeof window !== 'undefined') {
			optionsText = localStorage.getItem('spinwheelOptions') || 'Option 1\nOption 2\nOption 3\nOption 4\nOption 5\nOption 6';
			winners = JSON.parse(localStorage.getItem('spinwheelWinners') || '[]');
			eliminationMode = JSON.parse(localStorage.getItem('spinwheelEliminationMode') || 'false');
		}
		loading = false;
		
		// Initialize wheel after DOM is ready
		setTimeout(() => {
			if (wheelContainer) {
				wheel = new Wheel(wheelContainer, { 
					items,
					pointerAngle: 0
				});
				wheel.onRest = handleWinner;
			}
		}, 0);
	});

	$: if (wheel) {
		wheel.items = items;
		wheel.pointerAngle = 0;
		wheel.onRest = handleWinner;
	}

	function spin() {
		const velocity = Math.random() * 1000 + 1000;
		wheel.spin(velocity);
	}

	function resetWinners() {
		const allOptions = [...optionsText.split('\n').filter(line => line.trim()), ...winners];
		optionsText = allOptions.join('\n');
		winners = [];
		showResetModal = false;
	}

	function clearAll() {
		optionsText = '';
		winners = [];
		showResetModal = false;
	}
</script>

{#if loading}
	<div class="flex flex-1 items-center justify-center">
		<div class="text-center">
			<div class="animate-spin h-8 w-8 border-4 border-blue-500 border-t-transparent rounded-full mx-auto mb-2"></div>
			<p>Loading...</p>
		</div>
	</div>
{:else}
	<div class="flex flex-1">
		<div class="flex-1 flex flex-col items-center justify-center gap-4">
			<div class="flex relative flex-1 w-full">
				<div class="tick-mark"></div>
				<div class="relative flex-1 w-full" bind:this={wheelContainer}></div>
			</div>
			<label class="flex items-center gap-2">
				<input type="checkbox" bind:checked={eliminationMode} />
				Elimination Mode
			</label>
			<div class="flex gap-2">
				<button onclick={spin} class="spin-button">Spin!</button>
				<button onclick={() => showResetModal = true} class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600">Reset</button>
			</div>
		</div>
		<div class="w-64 p-4 border-l border-gray-300 flex flex-col gap-4">
			<textarea 
				bind:value={optionsText} 
				class="flex-1 p-2 border border-gray-300 rounded resize-none" 
				placeholder="Enter options, one per line"
			></textarea>
			{#if eliminationMode}
	            <h3 class="text-lg font-semibold">Winners:</h3>
				<textarea 
					value={winners.join('\n')} 
					readonly
					class="flex-1 p-2 border border-gray-300 rounded resize-none bg-gray-50" 
					placeholder="Winners will appear here"
				></textarea>
			{/if}
		</div>
	</div>
{/if}

<Modal open={showWinnerModal} onClose={closeWinnerModal}>
	<div class="text-center">
		<h2 class="text-2xl font-bold mb-4">🎉 Winner! 🎉</h2>
		<p class="text-xl mb-6">{currentWinner}</p>
		<button 
			class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
			onclick={closeWinnerModal}
		>
			OK
		</button>
	</div>
</Modal>

<Modal open={showResetModal} onClose={() => showResetModal = false}>
	<div class="text-center">
		<h2 class="text-xl font-bold mb-4">Reset Options</h2>
		<p class="mb-6">Choose how you want to reset:</p>
		<div class="flex flex-col gap-3">
			<button 
				class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
				onclick={resetWinners}
			>
				Reset Winners Back to Spinner
			</button>
			<button 
				class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600"
				onclick={clearAll}
			>
				Clear All Entries
			</button>
			<button 
				class="px-4 py-2 bg-gray-500 text-white rounded hover:bg-gray-600"
				onclick={() => showResetModal = false}
			>
				Cancel
			</button>
		</div>
	</div>
</Modal>

<style>
	.tick-mark {
		position: absolute;
		top: 2px;
		left: 50%;
		transform: translateX(-50%);
		width: 0;
		height: 0;
		border-left: 30px solid transparent;
		border-right: 30px solid transparent;
		border-top: 50px solid #333;
		z-index: 10;
	}

	.spin-button {
		padding: 0.5rem 1rem;
		background: #007bff;
		color: white;
		border: none;
		border-radius: 0.25rem;
		cursor: pointer;
	}

	.spin-button:hover {
		background: #0056b3;
	}
</style>