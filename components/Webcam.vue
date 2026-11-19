<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

const props = defineProps({
  mirror: {
    type: Boolean,
    default: false,
  },
  autoplay: {
    type: Boolean,
    default: true,
  },
})

const videoRef = ref<HTMLVideoElement | null>(null)
const stream = ref<MediaStream | null>(null)
const devices = ref<MediaDeviceInfo[]>([])
const selectedDeviceId = ref<string>(typeof localStorage !== 'undefined' ? localStorage.getItem('slidev-webcam-device') || '' : '')
const isMirrored = ref<boolean>(props.mirror)
const isStreaming = ref<boolean>(false)
const errorMessage = ref<string>('')

async function getDevices() {
  if (typeof navigator === 'undefined' || !navigator.mediaDevices?.enumerateDevices) return
  try {
    const allDevices = await navigator.mediaDevices.enumerateDevices()
    devices.value = allDevices.filter(d => d.kind === 'videoinput')
    if (!selectedDeviceId.value && devices.value.length > 0) {
      selectedDeviceId.value = devices.value[0].deviceId
    }
  } catch (err) {
    console.error('Failed to enumerate devices:', err)
  }
}

async function startCamera() {
  if (typeof navigator === 'undefined' || !navigator.mediaDevices?.getUserMedia) {
    errorMessage.value = 'お使いのブラウザはカメラのキャプチャをサポートしていません。'
    return
  }

  stopCamera()
  errorMessage.value = ''

  try {
    const constraints: MediaStreamConstraints = {
      video: selectedDeviceId.value
        ? { deviceId: { exact: selectedDeviceId.value }, width: { ideal: 1920 }, height: { ideal: 1080 } }
        : { width: { ideal: 1920 }, height: { ideal: 1080 } },
      audio: false,
    }
    const mediaStream = await navigator.mediaDevices.getUserMedia(constraints)
    stream.value = mediaStream
    if (videoRef.value) {
      videoRef.value.srcObject = mediaStream
      await videoRef.value.play()
    }
    isStreaming.value = true
    await getDevices()
  } catch (err: any) {
    console.error('Error starting camera:', err)
    errorMessage.value = err.message || 'カメラの起動に失敗しました。カメラの接続とブラウザの権限を確認してください。'
    isStreaming.value = false
  }
}

function stopCamera() {
  if (stream.value) {
    stream.value.getTracks().forEach(track => track.stop())
    stream.value = null
  }
  if (videoRef.value) {
    videoRef.value.srcObject = null
  }
  isStreaming.value = false
}

function toggleMirror() {
  isMirrored.value = !isMirrored.value
}

watch(selectedDeviceId, (newId) => {
  if (newId && typeof localStorage !== 'undefined') {
    localStorage.setItem('slidev-webcam-device', newId)
    startCamera()
  }
})

onMounted(async () => {
  if (props.autoplay) {
    await startCamera()
  } else {
    await getDevices()
  }
})

onUnmounted(() => {
  stopCamera()
})
</script>

<template>
  <div class="relative w-full max-w-3xl mx-auto flex flex-col items-center">
    <!-- ビデオ表示エリア -->
    <div class="relative w-full aspect-video bg-black rounded-2xl overflow-hidden shadow-2xl border-2 border-gray-700/50 flex items-center justify-center">
      <video
        ref="videoRef"
        autoplay
        playsinline
        muted
        class="w-full h-full object-contain transition-transform duration-200"
        :class="{ '-scale-x-100': isMirrored }"
      />

      <!-- エラーメッセージ -->
      <div v-if="errorMessage" class="absolute inset-0 flex flex-col items-center justify-center p-6 bg-black/80 text-white text-center">
        <div class="text-red-400 text-lg font-bold mb-2">📷 カメラを起動できません</div>
        <div class="text-xs text-gray-300 max-w-md">{{ errorMessage }}</div>
        <button
          @click="startCamera"
          class="mt-4 px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white text-sm font-bold rounded-lg transition"
        >
          再試行
        </button>
      </div>

      <!-- 停止中オーバーレイ -->
      <div v-else-if="!isStreaming" class="absolute inset-0 flex flex-col items-center justify-center bg-gray-900/90 text-white">
        <button
          @click="startCamera"
          class="px-5 py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white font-bold rounded-xl shadow-lg transition flex items-center gap-2"
        >
          <span>▶</span> カメラを開始
        </button>
      </div>
    </div>

    <!-- コントロールバー（カメラ選択・左右反転・停止/再開） -->
    <div class="mt-3 flex flex-wrap items-center justify-center gap-3 text-xs">
      <!-- カメラ切り替え -->
      <div v-if="devices.length > 1" class="flex items-center gap-1.5 bg-gray-100 dark:bg-gray-800 px-3 py-1.5 rounded-lg border border-gray-300 dark:border-gray-700 shadow-sm">
        <span class="text-gray-500 font-bold">カメラ:</span>
        <select
          v-model="selectedDeviceId"
          class="bg-transparent text-gray-800 dark:text-gray-200 font-medium outline-none cursor-pointer"
        >
          <option v-for="(dev, idx) in devices" :key="dev.deviceId" :value="dev.deviceId">
            {{ dev.label || `カメラ ${idx + 1}` }}
          </option>
        </select>
      </div>

      <!-- 反転トグル -->
      <button
        @click="toggleMirror"
        class="px-3 py-1.5 rounded-lg border font-semibold transition shadow-sm"
        :class="isMirrored ? 'bg-amber-100 dark:bg-amber-900/40 border-amber-400 text-amber-900 dark:text-amber-200' : 'bg-gray-100 dark:bg-gray-800 border-gray-300 dark:border-gray-700 text-gray-700 dark:text-gray-300'"
      >
        ↔ 左右反転: {{ isMirrored ? 'ON' : 'OFF' }}
      </button>

      <!-- 停止/再開 -->
      <button
        v-if="isStreaming"
        @click="stopCamera"
        class="px-3 py-1.5 rounded-lg bg-gray-100 dark:bg-gray-800 border border-gray-300 dark:border-gray-700 text-gray-700 dark:text-gray-300 hover:text-red-500 transition shadow-sm"
      >
        一時停止
      </button>
      <button
        v-else
        @click="startCamera"
        class="px-3 py-1.5 rounded-lg bg-emerald-600 text-white font-bold hover:bg-emerald-500 transition shadow-sm"
      >
        再開
      </button>
    </div>
  </div>
</template>
