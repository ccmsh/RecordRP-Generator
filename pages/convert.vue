<template>
  <div class="audio-converter">
    <h1>音声ファイルをOGGに変換</h1>
    <input type="file" accept=".mp3,.wav" @change="onFileChange" />
    <button :disabled="!file || loading" @click="convert">変換</button>
    <div v-if="loading">変換中...</div>
    <p v-if="error" style="color:red">{{ error }}</p>
    <a v-if="oggUrl" :href="oggUrl" download="converted.ogg">OGGファイルをダウンロード</a>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const file = ref<File | null>(null)
const oggUrl = ref<string | null>(null)
const error = ref('')
const loading = ref(false)

function onFileChange(e: Event) {
  const files = (e.target as HTMLInputElement).files
  file.value = files?.[0] ?? null
  oggUrl.value = null
  error.value = ''
}

async function convert() {
  if (!file.value) return
  loading.value = true
  error.value = ''
  oggUrl.value = null

  try {
    const ffmpegModule = await import('@ffmpeg/ffmpeg')
    const { createFFmpeg, fetchFile } = (ffmpegModule as any).default
    const ffmpeg = createFFmpeg({
      log: true,
      corePath: '/ffmpeg-core.js' // ←public配下に置いた場合
    })
    await ffmpeg.load()
    ffmpeg.FS('writeFile', file.value.name, await fetchFile(file.value))
    await ffmpeg.run('-i', file.value.name, 'output.ogg')
    const data = ffmpeg.FS('readFile', 'output.ogg')
    oggUrl.value = URL.createObjectURL(new Blob([data.buffer], { type: 'audio/ogg' }))
    ffmpeg.FS('unlink', file.value.name)
    ffmpeg.FS('unlink', 'output.ogg')
  } catch (e) {
    console.error(e)
    error.value = '変換に失敗しました'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.audio-converter {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #fafafa;
  text-align: center;
}
</style>