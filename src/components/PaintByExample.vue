<script setup lang="ts">
import {onMounted, reactive, ref} from 'vue'
import axios from "axios";

const image = reactive<{ width: number, height: number }>({ width: 500, height: 500 });
const originImageUrl = ref('');
const referenceImageUrl = ref('');
const generatedImageUrl = ref('');
const lineColor = ref('rgba(255,255,255,1)');
const backgroundImage = ref('');
const canvas = ref();
const context = ref();
const isDrawable = ref(false);  // クリック中の判定
const setBackgroundColor = (): void => {
  context.value.strokeStyle = 'rgba(255,255,255,1)';
  // context.value.fillStyle = 'rgba(0,0,0, 0.5)';
  // context.value.fillRect(0, 0, image.width, image.height);
}
onMounted(() => {
  canvas.value = document.querySelector('#canvas');
  context.value = canvas.value.getContext('2d');
  context.value.lineCap = 'round';
  context.value.lineJoin = 'round';
  context.value.lineWidth = 5;
  context.value.strokeStyle = lineColor.value;
});
const downloadImage = (): void => {
  var img = new Image();
  img.src = originImageUrl.value;
  backgroundImage.value = `background-image: url("${img.src}"); background-color:rgba(255,255,255,0.0); background-blend-mode:lighten;`;
  img.onload = () => {
    image.width = 512;
    image.height = 512;
  }
  context.value.strokeStyle = lineColor.value;
};
const onDragStart = (e: MouseEvent): void => {
  const posX = e.offsetX;
  const posY = e.offsetY;
  context.value.beginPath();
  context.value.lineTo(posX, posY);
  context.value.stroke();
  isDrawable.value = true;
};
const onDragEnd = (): void => {
  context.value.closePath();
  isDrawable.value = false;
};
const drawLine = (e: MouseEvent): void  => {
  if (!isDrawable.value) return;
  // ポインターの位置を格納
  const posX = e.offsetX;
  const posY = e.offsetY;
  context.value.lineTo(posX, posY);
  context.value.stroke();
};
const base64ImageData = ref('');
const save = (): void => {
  base64ImageData.value = canvas.value.toDataURL("image/jpeg");
};

const clear = (): void => {
  context.value.clearRect(0, 0, canvas.value.width, canvas.value.height);
  setBackgroundColor();
};

const changeLineWidth = (width: number): void => {
  context.value.lineWidth = width;
  context.value.strokeStyle = lineColor.value;
};

const generateImage = (): void => {
  axios
      .post(
          '/api/generate/paint-by-example',
          {
            origin: originImageUrl.value,
            mask: canvas.value.toDataURL("image/jpeg"),
            reference: referenceImageUrl.value
          },
          {
            headers: {
              'Content-Type': 'application/json;charset=utf-8'
            }
          }
      )
      .then((response) => {
        console.log(response.data);
        generatedImageUrl.value = response.data.image_url;
        console.log(generatedImageUrl);
      })
      .catch((error) => {
        console.log(error)
      })
      .finally(() => {})
};
</script>

<template>
  <main>
    <canvas
        id="canvas"
        :width="image.width" :height="image.height" :style="backgroundImage"
        @mousedown="onDragStart"
        @mousemove="drawLine"
        @mouseup="onDragEnd"
        @mouseout="onDragEnd"
    ></canvas>

    <input v-model="originImageUrl">
    <button @click="downloadImage">読み込む</button>
    <input v-model="referenceImageUrl">

    <div>
      太さ: <button @click="changeLineWidth(5)">小</button>
      太さ: <button @click="changeLineWidth(15)">中</button>
      太さ: <button @click="changeLineWidth(30)">大</button>
    </div>

    <button @click="clear">クリア</button>
    <button id="save" @click="generateImage">生成</button>

    <p v-if="base64ImageData">{{ base64ImageData }}</p>
    <img v-if="generatedImageUrl" :src="generatedImageUrl">
  </main>
</template>

<style scoped>
#canvas {
  border: solid 1px #000;
  box-sizing: border-box;
  background-image: url("https://github.com/Fantasy-Studio/Paint-by-Example/blob/main/examples/image/example_1.png?raw=true");
  background-position: center center;
  background-repeat: no-repeat;
  background-size: contain;
  width: 512px;
  height: 512px;
}
header {
  line-height: 1.5;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
