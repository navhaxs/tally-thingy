<script lang="ts">
	import { onMount } from 'svelte';
	import Tab from '$lib/Tab.svelte';
	import TallyView from '$lib/TallyView.svelte';
	import SpinWheelView from '$lib/SpinWheelView.svelte';

	let activeTab = 'Tally';
	let tabs = ['Tally', 'Spin the wheel', 'Timer', 'Soundboard', 'Dice roll'];

	onMount(() => {
		activeTab = localStorage.getItem('activeTab') || 'Tally';
	});

	function onTabChange(tab) {
		activeTab = tab;
		localStorage.setItem('activeTab', tab);
	}
</script>

<!-- <h1>TallyThingy</h1> -->

<Tab {activeTab} {tabs} {onTabChange}>
	{#if activeTab === 'Spin the wheel'}
		<SpinWheelView />
	{/if}
	{#if activeTab === 'Tally'}
		<TallyView />
	{/if}
	{#if activeTab === 'Timer'}
		<!-- Timer component will go here -->
		 <p>coming soon</p>
		 <p>game countdown timer, with sfx and vibration at time up</p>
	{/if}
	{#if activeTab === 'Soundboard'}
		<!-- Soundboard component will go here -->
		 <p>coming soon</p>
	{/if}
	{#if activeTab === 'Dice roll'}
		<!-- Dice roll component will go here -->
		 <p>coming soon</p>
	{/if}
</Tab>

<!-- <h3 class="mt-8">coming soon</h3>
<ul>
	<li>increment by .5 points</li>
	<li>Timer</li>
	<li>Soundboard</li>
	<li>Dice roll</li>
	<li>Spin the wheel</li>
</ul> -->
