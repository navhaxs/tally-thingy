<script>
    export let label = "Team";
    export let count = 0;
    export let onCountChange = () => {};

    function increment() {
        count++;
        onCountChange(count);
    }
    function decrement() {
        count--;
        onCountChange(count);
    }
</script>

<div class="flex items-center gap-4 my-4 h-14 pr-4">
    <input
        type="text"
        class="border border-gray-300 rounded px-2 py-1 text-2xl font-semibold h-full flex-1"
        bind:value={label}
    />
    <button class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300 text-xl font-bold h-full cursor-pointer" on:click={decrement}>-</button>
    <span class="text-2xl font-mono w-12 text-center">{count}</span>
    <button class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300 text-xl font-bold h-full cursor-pointer" on:click={increment}>+</button>
</div>