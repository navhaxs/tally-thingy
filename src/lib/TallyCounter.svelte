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
    <input 
        type="number" 
        class="text-2xl font-mono w-12 text-center border border-gray-300 rounded no-spinner h-full" 
        bind:value={count}
        on:input={() => onCountChange(count)}
    />
    <button class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300 text-xl font-bold h-full cursor-pointer" on:click={increment}>+</button>
</div>

<style>
    .no-spinner::-webkit-outer-spin-button,
    .no-spinner::-webkit-inner-spin-button {
        -webkit-appearance: none;
        margin: 0;
    }
    
    .no-spinner[type=number] {
        -moz-appearance: textfield;
    }
</style>