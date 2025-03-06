<template>
  <div class="home-container">
    <input @input="loadImage" type="file" id="img" name="img" accept="image/*" ref="imageInput" />
    <div class="image-container">
      <img ref="imageElement" id="main-image" />
      <img
        v-show="imageElement !== null"
        id="watermark"
        src="/public/watermark.png"
        ref="watermarkElement"
      />
    </div>
    <div class="export-buttons">
      <button @click="exportImage">Export</button>
      <button @click="exportImageToClipboard">Copy to clipboard</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'

const imageInput = ref<File[] | null>(null)
const imageElement = ref<HTMLImageElement | null>(null)
const watermarkElement = ref<HTMLImageElement | null>(null)
const isLoaded = ref(false)

function createCanvasImage(callback: (canvas: HTMLCanvasElement) => void) {
  console.log('exporting image')
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')

  if (!ctx) return
  if (!imageElement.value) return

  console.log('image', imageElement.value)

  const image = new Image()
  const watermark = new Image()

  image.onload = () => {
    canvas.width = image.width
    canvas.height = image.height

    const margin = 10
    const aspectRatio = watermark.width / watermark.height

    ctx.drawImage(image, 0, 0)
    ctx.drawImage(
      watermark,
      margin,
      image.height - (image.width * 0.3) / aspectRatio - margin,
      image.width * 0.3,
      (image.width * 0.3) / aspectRatio,
    )

    callback(canvas)
  }

  watermark.onload = () => {
    image.src = imageElement.value.src
  }

  watermark.src = watermarkElement.value?.src

  return canvas
}

function exportImageToClipboard() {
  createCanvasImage((canvas) => {
    if (!canvas) return
    canvas.toBlob((blob) => {
      const item = new ClipboardItem({ 'image/png': blob })
      navigator.clipboard.write([item])
    })
  })
}

function exportImage() {
  createCanvasImage((canvas) => {
    const link = document.createElement('a')
    link.download = 'image.png'
    link.href = canvas.toDataURL('image/png')
    link.click()
  })
}

function loadImage() {
  if (!imageInput.value) return
  if (!imageElement.value) return

  isLoaded.value = true
  const imageFile = imageInput.value.files[0]

  imageElement.value.src = URL.createObjectURL(imageFile)
}

onMounted(() => {
  imageElement.value.src = '/public/placeholder.png'
})
</script>

<style scoped>
.home-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 50%;
  margin: 0 auto;
  border: 1px solid #000;
  gap: 1rem;
  padding: 1rem;
}
.image-container {
  width: 100%;
  display: flex;
  justify-content: center;
  position: relative;
}

.export-buttons {
  display: flex;
  flex-direction: row;
  width: 100%;
  gap: 1rem;
}

.export-buttons button {
  flex: 1;
}

#watermark {
  position: absolute;
  bottom: 0;
  left: 0;
  z-index: 1;
  width: 300px;
}

#main-image {
  width: 100%;
  height: 100%;
}
</style>
