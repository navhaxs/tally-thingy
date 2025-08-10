<script>
  export let activeTab = '';
  export let tabs = [];
  export let onTabChange = () => {};

  function handleTabClick(tab) {
    onTabChange(tab);
  }
</script>

<div class="flex h-screen">
  <div class="flex flex-col border-r border-gray-300 p-2.5">
    {#each tabs as tab}
      <button
        class={`p-2.5 border-none cursor-pointer border-l-2 border-solid ${activeTab === tab ? 'bg-gray-100 border-blue-500' : 'border-transparent'}`}
        on:click={() => handleTabClick(tab)}
      >
        {tab}
      </button>
    {/each}
  </div>
  <div class="flex flex-1 p-2.5">
    <slot />
  </div>
</div>