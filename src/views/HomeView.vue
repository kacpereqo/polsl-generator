<template>
  <div
    class="home-container"
    @paste="pasteHandler"
    @drop.prevent="dropHandler"
    @dragenter.prevent="dragenterHandler"
    @dragover.prevent="dragoverHandler"
    @dragleave.prevent="dragleaveHandler"
  >
    <div class="input">
      <input type="file" ref="imageInput" @change="loadImage" accept="image" />
      <p v-show="!isDragging">Drop or paste image here</p>
      <p v-show="isDragging">Drop!!!!</p>
    </div>
    <div v-show="isLoaded" class="on-load">
      <hr />
      <div class="preview">
        <div class="image">
          <img ref="imageElement" />
          <img ref="watermarkElement" class="watermark" src="/watermark.png" />
        </div>
      </div>
      <div class="export">
        <button @click="exportImageToClipboard">Copy to clipboard</button>
        <button @click="exportImage">Download</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'

const imageInput = ref<HTMLInputElement | null>(null)
const imageElement = ref<HTMLImageElement | null>(null)
const watermarkElement = ref<HTMLImageElement | null>(null)
const isLoaded = ref(false)

const isDragging = ref(false)

function dragenterHandler() {
  isDragging.value = true
}

function dragoverHandler() {
  isDragging.value = true
}

function dragleaveHandler() {
  isDragging.value = false
}

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
    if (!imageElement.value) return
    image.src = imageElement.value.src
  }

  if (!watermarkElement.value) return
  watermark.src = watermarkElement.value?.src

  return canvas
}

function dropHandler(event: DragEvent) {
  event.preventDefault()

  if (!event.dataTransfer) return

  const files = event.dataTransfer.files
  if (!files.length) return

  if (!imageInput.value) return
  imageInput.value.files = files
  loadImage()

  isDragging.value = false
}

function pasteHandler(event: ClipboardEvent) {
  const items = event.clipboardData?.items
  if (!items) return

  for (let i = 0; i < items.length; i++) {
    if (items[i].type.indexOf('image') !== -1) {
      const blob = items[i].getAsFile()
      if (!blob) return

      const url = URL.createObjectURL(blob)
      if (!imageElement.value) return
      imageElement.value.src = url

      const name = blob.name
      const type = blob.type

      const list = new DataTransfer()
      list.items.add(new File([blob], name, { type }))
      if (!imageInput.value) return
      imageInput.value.files = list.files

      isLoaded.value = true

      // set imageInput to
    }
  }
}

function exportImageToClipboard() {
  createCanvasImage((canvas) => {
    if (!canvas) return
    canvas.toBlob((blob) => {
      if (!blob) return
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

  isLoaded.value = true
  if (!imageInput.value.files) return
  const imageFile = imageInput.value.files[0]

  if (!imageElement.value) return
  imageElement.value.src = URL.createObjectURL(imageFile)

  // set watermark margin to width 50% - image
}

onMounted(() => {
  const events = ['dragenter', 'dragover', 'dragleave', 'drop']
  events.forEach((eventName) => {
    document.body.addEventListener(eventName, (e) => e.preventDefault())
  })
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

.input {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.on-load {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  width: 100%;
}

.export {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.export button {
  padding: 0.5rem 1rem;
  background-color: #000;
  color: #fff;
  border: none;
  cursor: pointer;
  flex: 1;
}

.preview {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.image {
  position: relative;
}

.image img {
  width: 100%;
  max-height: 70vh;
  object-fit: contain;
}

hr {
  border: none;
  border-top: 1px solid #333;
  color: #333;
  overflow: visible;
  text-align: center;
  height: 5px;
  width: 100%;
}

hr::after {
  content: 'Preview';
  display: inline-block;
  position: relative;
  top: -0.7em;
  padding: 0 0.25em;
  background: #fff;
}

.watermark {
  position: absolute;
  bottom: 10px;
  left: 10px;
  max-width: 200px;
  z-index: 1;
}
</style>
