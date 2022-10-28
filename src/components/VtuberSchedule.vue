<script setup>
import { ref } from 'vue'

const image_type = ref('')
const image_data_url = ref('')

async function on_paste(e) {
  if (e.clipboardData.types.length == 0) return

  const item = e.clipboardData.items[0]
  image_type.value = item.type
  if ( item.type.startsWith('image/')) {
    const file = item.getAsFile()
    var reader = new FileReader()
    reader.onload = (e) => {
      const data_url = e.target.result
      image_data_url.value = data_url
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
    var reader = new FileReader()
    reader.onload = (e) => {
      const data_url = e.target.result
      image_data_url.value = data_url
    };
    reader.readAsDataURL(file)
  } else {
    alert('対応していないファイルです。')
  }
}
</script>

<template>
  <textarea class="source-input" placeholder="ここに画像を貼り付けるか、画像ファイルをドラッグ＆ドロップしてください。" v-on:paste="on_paste" v-on:drop.prevent="on_file_drop" v-on:dragover.prevent readonly />
  <p>画像の種類: {{ image_type }}</p>
  <img class="input-image" :src="image_data_url" />
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
</style>
