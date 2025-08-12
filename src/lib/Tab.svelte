<script>
  export let activeTab = '';
  export let tabs = [];
  export let onTabChange = () => {};

  let showMobileTabs = false;

  function handleTabClick(tab) {
    onTabChange(tab);
    showMobileTabs = false;
  }
</script>

<!-- Mobile Header -->
<div class="md:hidden fixed top-0 left-0 right-0 bg-white border-b border-gray-300 z-20 flex items-center justify-between p-4">
  <button class="text-xl" on:click={() => showMobileTabs = true}>☰</button>
  <h1 class="text-lg font-semibold">{activeTab}</h1>
  <div class="w-6"></div>
</div>

<!-- Mobile Tab Overlay -->
{#if showMobileTabs}
  <div class="md:hidden fixed inset-0 bg-white z-30 flex flex-col">
    <div class="flex items-center justify-between p-4 border-b border-gray-300">
      <h2 class="text-lg font-semibold">Select Tab</h2>
      <button class="text-xl" on:click={() => showMobileTabs = false}>✕</button>
    </div>
    <div class="flex-1 overflow-y-auto">
      {#each tabs as tab}
        <button
          class={`w-full p-4 text-left border-b border-gray-100 ${activeTab === tab ? 'bg-blue-50 text-blue-600' : ''}`}
          on:click={() => handleTabClick(tab)}
        >
          {tab}
        </button>
      {/each}
    </div>
  </div>
{/if}

<!-- Desktop Layout -->
<div class="flex h-screen">
  <div class="hidden md:flex flex-col border-r border-gray-300 p-2.5">
    {#each tabs as tab}
      <button
        class={`p-2.5 border-none cursor-pointer border-l-2 border-solid ${activeTab === tab ? 'bg-gray-100 border-blue-500' : 'border-transparent'}`}
        on:click={() => handleTabClick(tab)}
      >
        {tab}
      </button>
    {/each}
  </div>
  <div class="flex flex-1 p-2.5 pt-20 md:pt-2.5">
    <slot />
  </div>
</div>