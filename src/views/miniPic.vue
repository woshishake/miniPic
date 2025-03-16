<template>
  <div class="relative min-h-screen">
    <!-- Background with overlay -->
    <div class="absolute inset-0 bg-cover bg-center opacity-20" :style="{ backgroundImage: `url(${backgroundImage})` }"></div>
    
    <!-- Main content -->
    <div class="relative flex flex-col items-center justify-center min-h-screen px-4 py-8">
      <h1 class="text-4xl font-bold text-blue-600 mb-4">MiniPic Art Generator</h1>
      <p class="text-lg text-gray-700 mb-8">Upload a photo, apply filters, and convert it to pixel art.</p>
      
      <!-- Upload and controls section -->
      <div class="w-full max-w-3xl p-6 bg-white rounded-lg shadow-md mb-8">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <!-- Image upload section -->
          <div class="flex flex-col">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">Upload Image</h2>
            <label class="flex flex-col items-center justify-center w-full h-48 border-2 border-dashed border-gray-300 rounded-lg cursor-pointer bg-gray-50 hover:bg-gray-100 transition-colors mb-4">
              <div v-if="!originalImage" class="flex flex-col items-center justify-center text-center p-4">
                <svg class="w-1 h-1 text-gray-100 mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path>
                </svg>
                <p class="text-gray-500 mb-1">Click to upload or drag and drop</p>
                <p class="text-xs text-gray-400">PNG, JPG, GIF (MAX. 5MB)</p>
              </div>
              <div v-else class="w-full h-full flex items-center justify-center overflow-hidden">
                <img :src="originalImage" alt="Preview" class="max-w-full max-h-full object-contain" />
              </div>
              <input 
                type="file" 
                accept="image/*" 
                class="hidden" 
                @change="handleFileUpload" 
              />
            </label>
            <button 
              v-if="originalImage" 
              @click="clearImage" 
              class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition-colors self-center"
            >
              Remove Image
            </button>
          </div>
          
          <!-- Settings section -->
          <div class="flex flex-col">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">Settings</h2>
            
            <!-- Styles/Filters -->
            <div class="mb-6">
              <label class="block text-gray-700 mb-2">Filter Style:</label>
              <select 
                v-model="selectedFilter" 
                class="w-full p-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500"
              >
                <option value="none">None (Original)</option>
                <option value="pixel">Pixel Art</option>
                <option value="sepia">Sepia</option>
                <option value="grayscale">Grayscale</option>
                <option value="vintage">Vintage</option>
                <option value="edge">Edge Detection</option>
                <option value="emboss">Emboss</option>
                <option value="retro">Retro</option>
              </select>
            </div>
            
            <!-- Bit Size Selection -->
            <div class="mb-6">
              <label class="block text-gray-700 mb-2">Output Bit Size:</label>
              <div class="flex space-x-4">
                <label class="flex items-center">
                  <input type="radio" v-model="bitSize" value="8" class="mr-2">
                  <span>8-bit</span>
                </label>
                <label class="flex items-center">
                  <input type="radio" v-model="bitSize" value="16" class="mr-2">
                  <span>16-bit</span>
                </label>
                <label class="flex items-center">
                  <input type="radio" v-model="bitSize" value="64" class="mr-2">
                  <span>64-bit</span>
                </label>
              </div>
            </div>
            
            <!-- Pixelation options (only shown when pixel filter is selected) -->
            <div v-if="selectedFilter === 'pixel'" class="mb-6">
              <label class="block text-gray-700 mb-2">Pixel Size: {{ pixelSize }}px</label>
              <input 
                type="range" 
                min="2" 
                max="20" 
                v-model.number="pixelSize" 
                class="w-full"
              />
            </div>
            
            <!-- Color options (only for 8-bit and 16-bit) -->
            <div v-if="bitSize !== '64'" class="mb-6">
              <label class="block text-gray-700 mb-2">Color Palette:</label>
              <select 
                v-model="colorPalette" 
                class="w-full p-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500"
              >
                <option value="standard">Standard</option>
                <option value="nes">NES</option>
                <option value="gameboy">GameBoy</option>
                <option value="commodore">Commodore 64</option>
              </select>
            </div>
            
            <!-- Generate button -->
            <button 
              @click="generateFilteredImage" 
              class="mt-auto px-6 py-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors disabled:bg-blue-300 disabled:cursor-not-allowed"
              :disabled="!originalImage"
            >
              Generate Image
            </button>
          </div>
        </div>
      </div>
      
      <!-- Output image display -->
      <div v-if="outputImage" class="w-full max-w-3xl bg-white p-6 rounded-lg shadow-md">
        <h2 class="text-xl font-semibold text-gray-800 mb-4">Generated Image</h2>
        <div class="flex justify-center mb-6">
          <img :src="outputImage" alt="Generated Image" class="max-w-full max-h-96 object-contain border border-gray-200 rounded-lg" />
        </div>
        <div class="flex justify-center">
          <button 
            @click="downloadImage" 
            class="px-6 py-3 bg-green-500 text-white rounded-lg hover:bg-green-600 transition-colors"
          >
            Download Image
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import backgroundImage from '@/assets/background.png';

// Color palettes for different bit sizes
const COLOR_PALETTES = {
  standard: {
    '8': 256,
    '16': 65536,
  },
  nes: {
    '8': 64,
    '16': 64,
  },
  gameboy: {
    '8': 4,
    '16': 4,
  },
  commodore: {
    '8': 16,
    '16': 16,
  }
};

// Filter functions
const filters = {
  none: (ctx, canvas) => {
    // No filter, just return the original
    return ctx.getImageData(0, 0, canvas.width, canvas.height);
  },
  grayscale: (ctx, canvas) => {
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    
    for (let i = 0; i < data.length; i += 4) {
      const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
      data[i] = avg;     // R
      data[i + 1] = avg; // G
      data[i + 2] = avg; // B
    }
    
    return imageData;
  },
  sepia: (ctx, canvas) => {
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    
    for (let i = 0; i < data.length; i += 4) {
      const r = data[i];
      const g = data[i + 1];
      const b = data[i + 2];
      
      data[i] = Math.min(255, (r * 0.393) + (g * 0.769) + (b * 0.189));     // R
      data[i + 1] = Math.min(255, (r * 0.349) + (g * 0.686) + (b * 0.168)); // G
      data[i + 2] = Math.min(255, (r * 0.272) + (g * 0.534) + (b * 0.131)); // B
    }
    
    return imageData;
  },
  vintage: (ctx, canvas) => {
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    
    for (let i = 0; i < data.length; i += 4) {
      const r = data[i];
      const g = data[i + 1];
      const b = data[i + 2];
      
      data[i] = Math.min(255, r * 1.2 - 50);     // R
      data[i + 1] = Math.min(255, g * 0.9 + 10); // G
      data[i + 2] = Math.min(255, b * 0.8 + 30); // B
    }
    
    return imageData;
  },
  edge: (ctx, canvas) => {
    // Basic edge detection using a 3x3 Sobel operator
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const originalData = new Uint8ClampedArray(imageData.data);
    const data = imageData.data;
    const width = canvas.width;
    const height = canvas.height;
    
    // Grayscale first
    for (let i = 0; i < data.length; i += 4) {
      const avg = (originalData[i] + originalData[i + 1] + originalData[i + 2]) / 3;
      data[i] = avg;
      data[i + 1] = avg;
      data[i + 2] = avg;
    }
    
    // Simple edge detection
    for (let y = 1; y < height - 1; y++) {
      for (let x = 1; x < width - 1; x++) {
        const index = (y * width + x) * 4;
        
        // Compute the gradient approx with simple kernel
        const val00 = data[(y-1) * width * 4 + (x-1) * 4];
        const val01 = data[(y-1) * width * 4 + x * 4];
        const val02 = data[(y-1) * width * 4 + (x+1) * 4];
        
        const val10 = data[y * width * 4 + (x-1) * 4];
        const val12 = data[y * width * 4 + (x+1) * 4];
        
        const val20 = data[(y+1) * width * 4 + (x-1) * 4];
        const val21 = data[(y+1) * width * 4 + x * 4];
        const val22 = data[(y+1) * width * 4 + (x+1) * 4];
        
        const gx = val00 - val02 + 2 * val10 - 2 * val12 + val20 - val22;
        const gy = val00 + 2 * val01 + val02 - val20 - 2 * val21 - val22;
        
        const mag = Math.min(255, Math.sqrt(gx * gx + gy * gy));
        
        data[index] = mag;
        data[index + 1] = mag;
        data[index + 2] = mag;
      }
    }
    
    return imageData;
  },
  emboss: (ctx, canvas) => {
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const originalData = new Uint8ClampedArray(imageData.data);
    const data = imageData.data;
    const width = canvas.width;
    const height = canvas.height;
    
    for (let y = 1; y < height - 1; y++) {
      for (let x = 1; x < width - 1; x++) {
        const index = (y * width + x) * 4;
        const previous = ((y-1) * width + (x-1)) * 4;
        
        data[index] = 128 + originalData[index] - originalData[previous];
        data[index + 1] = 128 + originalData[index + 1] - originalData[previous + 1];
        data[index + 2] = 128 + originalData[index + 2] - originalData[previous + 2];
      }
    }
    
    return imageData;
  },
  retro: (ctx, canvas) => {
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    
    // Limited color palette for retro look
    const colors = [
      [0,0,0], [85,0,0], [170,0,0], [255,0,0],
      [0,85,0], [85,85,0], [170,85,0], [255,85,0],
      [0,170,0], [85,170,0], [170,170,0], [255,170,0],
      [0,255,0], [85,255,0], [170,255,0], [255,255,0]
    ];
    
    for (let i = 0; i < data.length; i += 4) {
      // Find closest color in the palette
      let closestColor = colors[0];
      let closestDistance = 999999;
      
      for (const color of colors) {
        const dr = data[i] - color[0];
        const dg = data[i+1] - color[1];
        const db = data[i+2] - color[2];
        const distance = dr*dr + dg*dg + db*db;
        
        if (distance < closestDistance) {
          closestDistance = distance;
          closestColor = color;
        }
      }
      
      data[i] = closestColor[0];
      data[i+1] = closestColor[1];
      data[i+2] = closestColor[2];
    }
    
    return imageData;
  }
};

export default {
  data() {
    return {
      backgroundImage,
      originalImage: null,
      outputImage: null,
      selectedFilter: 'pixel',
      pixelSize: 8,
      bitSize: '8',
      colorPalette: 'standard',
      fileType: 'image/png',
      fileName: 'filtered-image.png',
      processing: false
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;
      
      // Check file size (max 5MB)
      if (file.size > 5 * 1024 * 1024) {
        alert('File size exceeds 5MB limit. Please choose a smaller file.');
        return;
      }
      
      // Clear previous results
      this.outputImage = null;
      
      // Create object URL for display
      this.originalImage = URL.createObjectURL(file);
      
      // Extract file name for later use when downloading
      const nameParts = file.name.split('.');
      if (nameParts.length > 1) {
        nameParts.pop(); // Remove extension
        this.fileName = `${nameParts.join('.')}-filtered.png`;
      } else {
        this.fileName = 'filtered-image.png';
      }
    },
    
    clearImage() {
      this.originalImage = null;
      this.outputImage = null;
      // Reset file input
      const fileInput = document.querySelector('input[type="file"]');
      if (fileInput) {
        fileInput.value = '';
      }
    },
    
    applyBitDepthReduction(imageData, bitSize, colorPalette) {
      const data = imageData.data;
      let colors = COLOR_PALETTES[colorPalette][bitSize];
      
      // Default color depth reduction
      if (bitSize === '8') {
        // Reduce to 8-bit (256 colors)
        for (let i = 0; i < data.length; i += 4) {
          // For 8-bit, we typically reduce each channel to 3 bits R, 3 bits G, 2 bits B
          // That gives 8 * 8 * 4 = 256 colors
          if (colorPalette === 'standard') {
            data[i] = Math.floor(data[i] / 32) * 32;       // R (3 bits, 8 values)
            data[i + 1] = Math.floor(data[i + 1] / 32) * 32; // G (3 bits, 8 values)
            data[i + 2] = Math.floor(data[i + 2] / 64) * 64; // B (2 bits, 4 values)
          } else if (colorPalette === 'gameboy') {
            // GameBoy 4-color palette (light to dark)
            const brightness = (data[i] + data[i + 1] + data[i + 2]) / 3;
            let value;
            
            if (brightness > 192) {
              value = 255; // Lightest
            } else if (brightness > 128) {
              value = 192; // Light
            } else if (brightness > 64) {
              value = 96;  // Dark
            } else {
              value = 0;   // Darkest
            }
            
            // GameBoy greenish tint
            data[i] = Math.floor(value * 0.6);      // R
            data[i + 1] = Math.floor(value * 0.9);  // G
            data[i + 2] = Math.floor(value * 0.4);  // B
          }
          // Add other palette implementations here
        }
      } else if (bitSize === '16') {
        // Reduce to 16-bit (65,536 colors)
        for (let i = 0; i < data.length; i += 4) {
          if (colorPalette === 'standard') {
            // For 16-bit, typically 5 bits R, 6 bits G, 5 bits B
            data[i] = Math.floor(data[i] / 8) * 8;       // R (5 bits, 32 values)
            data[i + 1] = Math.floor(data[i + 1] / 4) * 4; // G (6 bits, 64 values)
            data[i + 2] = Math.floor(data[i + 2] / 8) * 8; // B (5 bits, 32 values)
          }
          // Add other palette implementations here
        }
      }
      // 64-bit is full color, no reduction needed
      
      return imageData;
    },
    
    applyPixelFilter(imageData, pixelSize, width, height) {
      const tempCanvas = document.createElement('canvas');
      const tempCtx = tempCanvas.getContext('2d');
      
      tempCanvas.width = width;
      tempCanvas.height = height;
      
      // Put the original image data in the temp canvas
      tempCtx.putImageData(imageData, 0, 0);
      
      // Clear the original canvas
      const pixelatedData = new ImageData(width, height);
      
      // Draw pixelated version
      for (let y = 0; y < height; y += pixelSize) {
        for (let x = 0; x < width; x += pixelSize) {
          // Get the color of the pixel at the start of each square
          const pixelData = tempCtx.getImageData(x, y, 1, 1).data;
          
          // Fill the square with that color
          for (let py = 0; py < pixelSize && y + py < height; py++) {
            for (let px = 0; px < pixelSize && x + px < width; px++) {
              const i = ((y + py) * width + (x + px)) * 4;
              pixelatedData.data[i] = pixelData[0];
              pixelatedData.data[i + 1] = pixelData[1];
              pixelatedData.data[i + 2] = pixelData[2];
              pixelatedData.data[i + 3] = pixelData[3];
            }
          }
        }
      }
      
      return pixelatedData;
    },
    
    generateFilteredImage() {
      if (!this.originalImage || this.processing) return;
      
      this.processing = true;
      
      const img = new Image();
      img.crossOrigin = 'Anonymous';
      img.src = this.originalImage;
      
      img.onload = () => {
        // Create a canvas to process the image
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        
        // Set canvas size based on input image
        const aspectRatio = img.width / img.height;
        let targetWidth, targetHeight;
        
        if (aspectRatio >= 1) {
          // Landscape or square
          targetWidth = 512;
          targetHeight = Math.round(targetWidth / aspectRatio);
        } else {
          // Portrait
          targetHeight = 512;
          targetWidth = Math.round(targetHeight * aspectRatio);
        }
        
        canvas.width = targetWidth;
        canvas.height = targetHeight;
        
        // Draw image on canvas
        ctx.drawImage(img, 0, 0, targetWidth, targetHeight);
        
        // Apply the selected filter (except pixel filter, which comes after bit depth)
        let processedImageData;
        if (this.selectedFilter !== 'pixel') {
          processedImageData = filters[this.selectedFilter](ctx, canvas);
        } else {
          processedImageData = ctx.getImageData(0, 0, targetWidth, targetHeight);
        }
        
        // Apply bit depth reduction based on settings
        processedImageData = this.applyBitDepthReduction(
          processedImageData, 
          this.bitSize, 
          this.colorPalette
        );
        
        // Apply pixelation filter if selected
        if (this.selectedFilter === 'pixel') {
          processedImageData = this.applyPixelFilter(
            processedImageData,
            this.pixelSize,
            targetWidth,
            targetHeight
          );
        }
        
        // Put the processed image data back on the canvas
        ctx.putImageData(processedImageData, 0, 0);
        
        // Convert canvas to data URL and set as output image
        this.outputImage = canvas.toDataURL(this.fileType);
        this.processing = false;
      };
      
      img.onerror = () => {
        alert('Error loading image. Please try another file.');
        this.processing = false;
      };
    },
    
    downloadImage() {
      if (!this.outputImage) return;
      
      const downloadLink = document.createElement('a');
      downloadLink.href = this.outputImage;
      downloadLink.download = this.fileName;
      
      document.body.appendChild(downloadLink);
      downloadLink.click();
      document.body.removeChild(downloadLink);
    }
  }
};
</script>

<style scoped>
/* Style overrides for range inputs */
input[type="range"] {
  -webkit-appearance: none;
  height: 8px;
  border-radius: 4px;
  background: #d1d5db;
  outline: none;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
}

input[type="range"]::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
}

/* Animation for loading state */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
</style>