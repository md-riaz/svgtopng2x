<script lang="ts">
  import { onMount } from 'svelte';
  
  export let svgContent: string = '';
  export let scale: number = 1;
  
  let previewContainer: HTMLDivElement;
  let { width = $state(0), height = $state(0) }
  
  $effect(() => {
    if (svgContent && previewContainer) {
      const svg = previewContainer.querySelector('svg');
      if (svg) {
        width = svg.viewBox.baseVal.width || svg.width.baseVal.value;
        height = svg.viewBox.baseVal.height || svg.height.baseVal.value;
      }
    } else {
      width = 0;
      height = 0;
    }
  });
</script>

<div 
  bind:this={previewContainer}
  class="mt-6 border rounded-lg p-6 bg-gray-50 min-h-[200px] flex items-center justify-center overflow-auto"
>
  {#if svgContent}
    <div style="transform: scale({scale}); transform-origin: center; transition: transform 0.2s ease-out;">
      {@html svgContent}
    </div>
  {:else}
    <p class="text-gray-500 italic">SVG preview will appear here</p>
  {/if}
</div>

<div class="mt-2 text-sm text-gray-500 text-center">
  {#if width && height}
    Original size: {width}x{height}px
  {/if}
</div>