<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  
  const dispatch = createEventDispatcher<{
    svgContent: { content: string };
    error: { message: string };
  }>();
  
  let { isDragging = $state(false) }
  
  async function handleFileSelect(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    
    if (!file) return;
    
    if (!file.type.includes('svg')) {
      dispatch('error', { message: 'Please select an SVG file' });
      return;
    }
    
    try {
      const content = await file.text();
      const parser = new DOMParser();
      const doc = parser.parseFromString(content, 'image/svg+xml');
      
      if (doc.getElementsByTagName('parsererror').length > 0) {
        dispatch('error', { message: 'Invalid SVG file' });
        return;
      }
      
      dispatch('svgContent', { content });
    } catch (e) {
      dispatch('error', { message: 'Error reading file' });
    }
  }
</script>

<div
  class="relative border-2 border-dashed rounded-lg p-6 transition-colors"
  class:border-indigo-300={!isDragging}
  class:border-indigo-500={isDragging}
  class:bg-indigo-50={isDragging}
  on:dragenter={() => isDragging = true}
  on:dragleave={() => isDragging = false}
  on:dragover|preventDefault
>
  <input
    type="file"
    accept=".svg"
    on:change={handleFileSelect}
    class="absolute inset-0 w-full h-full opacity-0 cursor-pointer"
  />
  <div class="text-center">
    <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48">
      <path d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
    </svg>
    <p class="mt-1 text-sm text-gray-600">
      Drop your SVG file here or click to browse
    </p>
  </div>
</div>