<script setup lang="ts">
import {onMounted, reactive, ref} from 'vue'

const image = reactive<{ width: number, height: number }>({ width: 500, height: 500 });
const imageUrl = ref('');
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
  img.src = imageUrl.value;
  backgroundImage.value = `background-image: url("${img.src}"); background-color:rgba(255,255,255,0.5); background-blend-mode:lighten;`;
  img.onload = () => {
    image.width = img.width;
    image.height = img.height;
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

    <input v-model="imageUrl">
    <button @click="downloadImage">読み込む</button>

    <div>
      太さ: <button @click="changeLineWidth(5)">小</button>
      太さ: <button @click="changeLineWidth(10)">中</button>
      太さ: <button @click="changeLineWidth(20)">大</button>
    </div>

    <button @click="clear">クリア</button>
    <button id="save" @click="save">保存</button>

    <p v-if="base64ImageData">{{ base64ImageData }}</p>
  </main>
</template>

<style scoped>
#canvas {
  border: solid 1px #000;
  box-sizing: border-box;
  background-image: url("https://cdn.grail.bz/images/goods/d/kz265/kz265_v1.jpg");
  background-repeat: no-repeat;
  background-size: 100% auto;
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
