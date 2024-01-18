<template>
 <label class="block">
    <span class="sr-only">选择图片</span>
    
    <input
      type="file"
      class="block w-full text-sm text-slate-500
        file:mr-4 file:py-2 file:px-4
        file:rounded-full file:border-0
        file:text-sm file:font-semibold
        file:bg-blue-50 file:text-blue-700
        hover:file:bg-blue-100
      "
      id="imageInput"
      @change="onImageChange">
  </label>  
 
  <div class="flex space-x-8 mb-4 mt-4" >
    <div class="relative flex items-center mr-2">
      <input
        type="number"
        v-model="radius"
        id="radiusInput"
        class="w-full px-3 py-2 text-gray-700 border rounded appearance-none focus:outline-none focus:shadow-outline"
        placeholder="请输入圆角半径"
      />
      <span class="text-gray-500 text-lm ml-2">px</span>
      <div v-if="radiusError" class="text-red-500 text-sm mt-1">{{ radiusError }}</div>
    </div>
    <button @click="generate" class="bg-blue-500 text-white px-4 py-2 rounded w-24 ml-2 mr-2">生成</button>
  
    <button @click="download" class="bg-green-500 text-white px-4 py-2 rounded w-24 ml-2 mr-2">下载</button> 

  </div>
 <!-- Modal for displaying errors -->
 <div v-if="errorModal" class="fixed top-0 left-0 w-full h-full flex items-center justify-center bg-black bg-opacity-50">
    <div class="bg-white p-4 rounded shadow">
      <p class="text-red-500">{{ radiusError }}</p>
      <button @click="closeErrorModal" class="mt-2 bg-blue-500 text-white px-4 py-2 rounded">确定</button>
    </div>
  </div>
  <canvas id="outputCanvas" width="500" height="500" class="border mt-2"></canvas>

</template>

<script setup>
import { ref, onMounted } from 'vue'

const radius = ref('');
const radiusError = ref('');
const errorModal = ref(false);

const resultCanvas = ref(null)

onMounted(() => {
  const canvas = document.getElementById('outputCanvas')
  const ctx = canvas.getContext('2d')

  drawCheckerboardBackground(ctx, canvas.width, canvas.height)
})

function generate() {


  const radius = document.getElementById('radiusInput').value

  const inputRadius = parseFloat(radius);

if (isNaN(inputRadius) || inputRadius < 0) {
  radiusError.value = '请输入有效的圆角半径（大于等于0）';
  errorModal.value = true;

  return;
}

  const imageInput = document.getElementById('imageInput')
  const image = new Image()
  image.src = URL.createObjectURL(imageInput.files[0])

  image.onload = () => {
    const tempCanvas = document.createElement('canvas')
    const tempCtx = tempCanvas.getContext('2d')

    tempCanvas.width = image.width
    tempCanvas.height = image.height

    resultCanvas.value = tempCanvas
    
    drawRoundedImage(tempCtx, image, radius)

    const canvas = document.getElementById('outputCanvas')
    const ctx = canvas.getContext('2d')

    const iw = tempCanvas.width
    const ih = tempCanvas.height
    const x = (canvas.width - iw) / 2
    const y = (canvas.height - ih) / 2

    ctx.clearRect(0, 0, canvas.width, canvas.height)

    drawCheckerboardBackground(ctx, canvas.width, canvas.height)

    ctx.drawImage(tempCanvas, x, y)
  }
}

function drawRoundedImage(ctx, image, radius) {
  ctx.beginPath()
  ctx.moveTo(radius, 0)
  ctx.arcTo(image.width, 0, image.width, image.height, radius)
  ctx.arcTo(image.width, image.height, 0, image.height, radius)
  ctx.arcTo(0, image.height, 0, 0, radius) 
  ctx.arcTo(0, 0, image.width, 0, radius)
  ctx.closePath()
  ctx.clip()

  ctx.drawImage(image, 0, 0, image.width, image.height)
}

function download() {
  const link = document.createElement('a')
  link.download = 'image.png'
  link.href = resultCanvas.value.toDataURL()
  link.click()  
}

function onImageChange() {
  previewImage()
}

function previewImage() {
  const canvas = document.getElementById('outputCanvas')
  const ctx = canvas.getContext('2d')

  const image = new Image()
  image.onload = () => {
    const iw = image.width
    const ih = image.height
    const x = (canvas.width - iw) / 2
    const y = (canvas.height - ih) / 2

    ctx.clearRect(0, 0, canvas.width, canvas.height)
        
    drawCheckerboardBackground(ctx, canvas.width, canvas.height)

    ctx.drawImage(image, x, y, iw, ih)
  }

  image.src = URL.createObjectURL(document.getElementById('imageInput').files[0])
}

function drawCheckerboardBackground(ctx, width, height) {
  const tileSize = 20
  const colors = ['#fff', '#eee']

  for(let x = 0; x < width; x += tileSize) {
    for(let y = 0; y < height; y += tileSize) {
      const colorIndex = (x / tileSize + y / tileSize) % 2
      ctx.fillStyle = colors[colorIndex]  
      ctx.fillRect(x, y, tileSize, tileSize)
    }
  }
}
function closeErrorModal() {
  errorModal.value = false;
}
</script>

<style scoped>
@tailwind base;
@tailwind components;
@tailwind utilities;
</style>