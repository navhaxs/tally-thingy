<script lang="ts">
    import TallyCounter from '$lib/TallyCounter.svelte';
    import { dndzone } from 'svelte-dnd-action';
    import Modal from '$lib/Modal.svelte';

    interface Tally {
        id: number;
        label: string;
        count: number;
    }

    const STORAGE_KEY = 'tallyThingyState';

    function getDefaultTallies(): Tally[] {
        return [];
    }

    function saveState() {
        if (typeof window !== 'undefined') {
            localStorage.setItem(
                STORAGE_KEY,
                JSON.stringify({ tallies, nextId, newTeamName })
            );
        }
    }

    let tallies: Tally[] = getDefaultTallies();
    let nextId: number = 3;
    let newTeamName: string = "";
    let pendingRemoveId: number | null = null;
    let loading: boolean = true;
    let pendingReset: boolean = false;
    let dragDisabled = true;

    let showRemoveModal = false;
    let showResetModal = false;
    let modalTally: Tally | null = null;
    let isLocked = true;

    import { onMount } from 'svelte';
    onMount(() => {
        if (typeof window !== 'undefined') {
            const raw = localStorage.getItem(STORAGE_KEY);
            if (raw) {
                try {
                    const { tallies: t, nextId: n, newTeamName: nt } = JSON.parse(raw);
                    tallies = t;
                    nextId = n;
                    newTeamName = nt || "";
                } catch {}
            }
        }
        loading = false;
    });

    function updateCount(index, newCount) {
        tallies[index].count = newCount;
        saveState();
    }

    function addTeam() {
        const name = newTeamName.trim() || `Team ${tallies.length + 1}`;
        tallies = [...tallies, { id: nextId++, label: name, count: 0 }];
        newTeamName = "";
        saveState();
    }

    function startDrag() {
        dragDisabled = false;
    }
    function stopDrag() {
        dragDisabled = true;
    }

    function handleConsider(evt: CustomEvent) {
        tallies = evt.detail.items;
    }
    function handleFinalize(evt: CustomEvent) {
        tallies = evt.detail.items;
        dragDisabled = true;
    }

    function confirmRemove(id) {
        tallies = tallies.filter(t => t.id !== id);
        pendingRemoveId = null;
        saveState();
    }

    function cancelRemove() {
        pendingRemoveId = null;
    }

    function resetApp() {
        if (typeof window !== 'undefined') {
            localStorage.removeItem(STORAGE_KEY);
        }
        tallies = getDefaultTallies();
        nextId = 3;
        newTeamName = "";
        pendingRemoveId = null;
    }

    function confirmReset() {
        resetApp();
        pendingReset = false;
    }

    function cancelReset() {
        pendingReset = false;
    }

    function openRemoveModal(tally: Tally) {
        modalTally = tally;
        showRemoveModal = true;
    }
    function closeRemoveModal() {
        showRemoveModal = false;
        modalTally = null;
    }
    function confirmRemoveModal() {
        if (modalTally) confirmRemove(modalTally.id);
        closeRemoveModal();
    }

    function openResetModal() {
        showResetModal = true;
    }
    function closeResetModal() {
        showResetModal = false;
    }
    function confirmResetModal() {
        confirmReset();
        closeResetModal();
    }
</script>

{#if loading}
    <div class="flex flex-col items-center justify-center h-screen">
        <svg class="animate-spin h-12 w-12 text-blue-500 mb-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4a4 4 0 00-4 4H4z"></path>
        </svg>
        <span class="text-gray-500 text-lg">Loading...</span>
    </div>
{:else}
    <div class="p-8 max-w-3xl mx-auto">

        <div class="flex items-center gap-2 mb-6">
            <input
                type="text"
                class="border border-gray-300 rounded px-2 py-1 w-48"
                placeholder="New team name"
                bind:value={newTeamName}
                on:keydown={(e) => e.key === 'Enter' && addTeam()}
            />
            <button
                class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 font-bold cursor-pointer"
                on:click={addTeam}
            >
                Add Team
            </button>
            <button
                class="px-4 py-2 bg-yellow-500 text-white rounded hover:bg-yellow-600 font-bold cursor-pointer"
                on:click={() => isLocked = !isLocked}
            >
                {isLocked ? '🔒' : '🔓'}
            </button>
        </div>

        <hr class="my-6 border-gray-300" />

        <ul
            use:dndzone={{ items: tallies, dragDisabled, flipDurationMs: 200 }}
            on:consider={handleConsider}
            on:finalize={handleFinalize}
            class="space-y-2"
        >
            {#each tallies as tally, i (tally.id)}
                <li class="flex items-stretch gap-2 bg-white rounded shadow px-2 py-1 min-h-[56px]" animate:flip={{ duration: 200 }}>
                    {#if !isLocked}
                        <span
                            class="handle cursor-grab text-2xl select-none pr-2 flex items-center"
                            title="Drag to reorder"
                            on:mousedown={startDrag}
                            on:touchstart={startDrag}
                            on:mouseup={stopDrag}
                            on:touchend={stopDrag}
                            style="user-select: none;"
                        >☰</span>
                    {/if}
                    <TallyCounter
                        label={tally.label}
                        count={tally.count}
                        onCountChange={(c) => updateCount(i, c)}
                    />
                    {#if !isLocked}
                        <button
                            class="ml-auto px-4 bg-red-500 text-white rounded hover:bg-red-600 font-bold cursor-pointer flex items-center"
                            on:click={() => openRemoveModal(tally)}
                        >
                            Remove
                        </button>
                    {/if}
                </li>
            {/each}
        </ul>

        {#if !isLocked}
            <div class="relative inline-block mt-8">
                <button
                    class="ml-4 px-4 py-2 bg-gray-500 text-white rounded hover:bg-gray-600 font-bold cursor-pointer"
                    on:click={openResetModal}
                >
                    Reset All
                </button>
            </div>
        {/if}

        <Modal open={showRemoveModal && !!modalTally} onClose={closeRemoveModal}>
            <p>Remove <strong>{modalTally?.label}</strong>?</p>
            <div class="flex gap-2 mt-4 justify-end">
                <button class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 cursor-pointer" on:click={confirmRemoveModal}>Yes</button>
                <button class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400 cursor-pointer" on:click={closeRemoveModal}>No</button>
            </div>
        </Modal>

        <Modal open={showResetModal} onClose={closeResetModal}>
            <p>Reset all teams and scores?</p>
            <div class="flex gap-2 mt-4 justify-end">
                <button class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 cursor-pointer" on:click={confirmResetModal}>Yes</button>
                <button class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400 cursor-pointer" on:click={closeResetModal}>No</button>
            </div>
        </Modal>
    </div>
{/if}