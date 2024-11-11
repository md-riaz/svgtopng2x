<script lang="ts">
  import FilePicker from './components/FilePicker.svelte';
  import Preview from './components/Preview.svelte';
  import ScaleControl from './components/ScaleControl.svelte';
  
  let { svgContent = $state('') }
  let { scale = $state(1) }
  let { error = $state('') }
  let { originalWidth = $state(0) }
  let { originalHeight = $state(0) }
  
  function handleSvgContent(event: CustomEvent<{ content: string }>) {
    svgContent = event.detail.content;
    error = '';
    
    // Parse SVG dimensions
    const parser = new DOMParser();
    const doc = parser.parseFromString(svgContent, 'image/svg+xml');
    const svg = doc.querySelector('svg');
    if (svg) {
      originalWidth = svg.viewBox.baseVal.width || svg.width.baseVal.value;
      originalHeight = svg.viewBox.baseVal.height || svg.height.baseVal.value;
    }
  }
  
  function handleError(event: CustomEvent<{ message: string }>) {
    error = event.detail.message;
    svgContent = '';
    originalWidth = 0;
    originalHeight = 0;
  }
  
  function handleScaleChange(event: CustomEvent<{ scale: number }>) {
    scale = event.detail.scale;
  }
  
  async function convertToPng() {
    if (!svgContent) return;
    
    try {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      if (!ctx) throw new Error('Canvas context not available');
      
      const blob = new Blob([svgContent], { type: 'image/svg+xml' });
      const url = URL.createObjectURL(blob);
      
      const img = new Image();
      img.src = url;
      
      await new Promise((resolve, reject) => {
        img.onload = resolve;
        img.onerror = reject;
      });
      
      canvas.width = originalWidth * scale;
      canvas.height = originalHeight * scale;
      
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      
      const pngUrl = canvas.toDataURL('image/png');
      const link = document.createElement('a');
      link.download = `converted_${scale}x.png`;
      link.href = pngUrl;
      link.click();
      
      URL.revokeObjectURL(url);
    } catch (e) {
      error = 'Error converting SVG to PNG';
    }
  }
</script>

<div class="max-w-3xl mx-auto p-6 bg-white rounded-lg shadow-lg">
  <h2 class="text-2xl font-bold text-gray-900 mb-6">SVG to PNG Converter</h2>
  
  <FilePicker
    on:svgContent={handleSvgContent}
    on:error={handleError}
  />
  
  {#if error}
    <div class="bg-red-50 border-l-4 border-red-400 p-4 my-4">
      <p class="text-red-700">{error}</p>
    </div>
  {/if}
  
  <ScaleControl
    {scale}
    {originalWidth}
    {originalHeight}
    on:scaleChange={handleScaleChange}
  />
  
  <Preview {svgContent} {scale} />
  
  <button
    on:click={convertToPng}
    disabled={!svgContent}
    class="w-full max-w-xs mx-auto mt-6 bg-indigo-600 text-white py-2 px-4 rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:bg-gray-300 disabled:cursor-not-allowed transition-colors"
  >
    Convert to PNG ({scale}x)
  </button>
</div>