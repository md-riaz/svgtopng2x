<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  
  export let scale = 1;
  export let originalWidth = 0;
  export let originalHeight = 0;
  
  const dispatch = createEventDispatcher<{
    scaleChange: { scale: number };
  }>();
  
  const presetScales = [1, 2, 3];
  
  function handleScaleChange(newScale: number) {
    dispatch('scaleChange', { scale: newScale });
  }
  
  $derived scaledWidth = Math.round(originalWidth * scale);
  $derived scaledHeight = Math.round(originalHeight * scale);
</script>

<div class="mt-6 space-y-4">
  <div class="flex items-center justify-between">
    <div>
      <label class="block text-sm font-medium text-gray-700">Scale Factor</label>
      <div class="mt-2 flex items-center gap-2">
        {#each presetScales as preset}
          <button
            class="px-3 py-1 rounded-md text-sm font-medium transition-colors"
            class:bg-indigo-600={scale === preset}
            class:text-white={scale === preset}
            class:bg-gray-100={scale !== preset}
            class:text-gray-700={scale !== preset}
            class:hover:bg-indigo-500={scale !== preset}
            on:click={() => handleScaleChange(preset)}
          >
            {preset}x
          </button>
        {/each}
      </div>
    </div>
    
    <div class="text-right">
      <span class="block text-sm font-medium text-gray-700">Resolution</span>
      {#if originalWidth && originalHeight}
        <span class="text-sm text-gray-500">
          {originalWidth}x{originalHeight} → {scaledWidth}x{scaledHeight}
        </span>
      {:else}
        <span class="text-sm text-gray-400 italic">No image loaded</span>
      {/if}
    </div>
  </div>
  
  <div>
    <input
      type="range"
      min="0.1"
      max="5"
      step="0.1"
      value={scale}
      on:input={(e) => handleScaleChange(+e.currentTarget.value)}
      class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-indigo-600"
    />
    <div class="flex justify-between text-xs text-gray-500 mt-1">
      <span>0.1x</span>
      <span>5x</span>
    </div>
  </div>
</div>