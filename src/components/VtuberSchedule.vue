<script setup>
import { ref } from 'vue'
import { createWorker } from 'tesseract.js';

const image_type = ref('')
const image_data_url = ref('')
const text_in_image = ref('')

function on_paste(e) {
  if (e.clipboardData.types.length == 0) return

  const item = e.clipboardData.items[0]
  image_type.value = item.type
  if ( item.type.startsWith('image/')) {
    const file = item.getAsFile()
    const reader = new FileReader()
    reader.onload = async (e) => {
      const data_url = e.target.result
      image_data_url.value = data_url
      text_in_image.value = '読込中'

      const text = await ocr(data_url)
      console.log(text)
      text_in_image.value = text
    }
    reader.readAsDataURL(file)
  } else {
    alert('対応していないデータです。')
  }
}

function on_file_drop(e) {
  if (e.dataTransfer.files.length == 0) return

  const file = e.dataTransfer.files[0]
  image_type.value = file.type

  if (file.type.startsWith('image/')) {
    const reader = new FileReader()
    reader.onload = async (e) => {
      const data_url = e.target.result
      image_data_url.value = data_url
      text_in_image.value = '読込中'

      const text = await ocr(data_url)
      console.log(text)
      text_in_image.value = text
    };
    reader.readAsDataURL(file)
  } else {
    alert('対応していないファイルです。')
  }
}

async function ocr(image) {
  const ocr_worker = createWorker({
    logger: m => console.log(m)
  })
  await ocr_worker.load()
  await ocr_worker.loadLanguage('jpn+eng')
  await ocr_worker.initialize('jpn+eng')
  const { data: { text } } = await ocr_worker.recognize(image)
  await ocr_worker.terminate()

  return text
}
</script>

<template>
  <textarea class="source-input" placeholder="ここに画像を貼り付けるか、画像ファイルをドラッグ＆ドロップしてください。" v-on:paste="on_paste" v-on:drop.prevent="on_file_drop" v-on:dragover.prevent readonly />
  <p>画像の種類: {{ image_type }}</p>
  <img class="input-image" :src="image_data_url" />
  <div class="text-in-image">{{ text_in_image }}</div>
</template>

<style scoped>
.source-input {
  text-align: center;
  resize: none;
  height: 100px;
}

.input-image {
  width: 500px;
}

.text-in-image {
  white-space: pre-wrap;
}
</style>
