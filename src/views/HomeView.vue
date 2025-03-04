
<template>
  <div class="home-container">
    <form @submit.prevent="loadImage">
      <label for="img"></label>
      <input type="file" id="img" name="img" accept="image/*" ref="imageInput"> 
      <button type="submit">Load Image</button>
    </form>
    <button @click="exportImage">Export</button>
    <div class="image-container">
      <img src=""` ref="imageElement" />
       <img v-show="imageElement===null" src="/public/watermark.png" alt="Image" id="watermark" ref="watermarkElement" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const imageInput = ref(null)
const imageElement = ref(null)
const watermarkElement = ref(null)

function exportImage(){
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  const image = imageElement.value
  const watermark = watermarkElement.value

  canvas.width = image.width
  canvas.height = image.height

  ctx.drawImage(image, 0, 0)
  ctx.drawImage(watermark,0, image.height - watermark.height) 

  const link = document.createElement('a')
  link.download = 'image.png'
  link.href = canvas.toDataURL('image/png')
  link.click()
}

function loadImage(event: Event) {
  const image = imageInput.value.files[0]
  imageElement.value.src = URL.createObjectURL(image)

  console.log(image)
  }
</script>

<style scoped>
.image-container{
  width: 50%;
  margin: 0 auto;
  height: 100%;
  display: flex;
  position: relative;
  flex-direction: column;
}

#watermark{
  position: absolute;
  bottom: 0;
  left: 0;
  z-index: 1;
}
</style>
