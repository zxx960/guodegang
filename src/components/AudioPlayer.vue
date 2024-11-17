<template>
  <div ref="audioPlayerContainer"
    class="w-full px-2 z-20 bottom-0 fixed bg-white border-t border-gray-200 lg:px-6 md:px-0 h-20 items-center flex shadow-lg">
    <audio ref="audioElement" :src="audioSrc" preload="metadata"></audio>

    <!-- 播放/暂停按钮 -->
    <button class="p-2 rounded-full transition-transform" @click="togglePlay">
      <img :src="props.isPlaying ? pause : play" :alt="props.isPlaying ? 'pause' : 'play'" class="w-7 h-7">
    </button>

    <!-- 时间显示 -->
    <div class="flex items-center text-sm text-gray-600">
      <span>{{ isDragging ? calculateTime(previewTime) : currentTimeFormatted }}</span>
      <span>/</span>
      <span>{{ durationFormatted }}</span>
    </div>

    <!-- 进度条容器 -->
    <div class="flex flex-1 mx-2 md:mx-6 relative" ref="progressContainer">
      <div class="w-full h-8 absolute -top-2 cursor-pointer" 
           @click="onProgressBarClick"
           @touchstart.prevent="onTouchStart"
           @touchmove.prevent="onTouchMove"
           @touchend.prevent="onTouchEnd"></div>
      <input 
        type="range" 
        v-model="seekValue" 
        :max="duration" 
        min="0"
        class="w-full h-1 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500" 
        @mousedown="startDragging"
        @mousemove="onDragging"
        @mouseup="stopDragging"
        @mouseleave="stopDragging"
        :style="progressBarStyle"
      />
    </div>

    <!-- 音量控制 -->
    <button class="p-2 rounded-full transition-transform" @click="toggleMute">
      <img :src="isMuted ? audioMuted : audio" :alt="isMuted ? 'audio-muted' : 'audio'" class="w-6 h-6">
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, defineProps, nextTick, defineEmits } from 'vue'
import play from '../assets/play.png'
import pause from '../assets/pause.png'
import audio from '../assets/audio.png'
import audioMuted from '../assets/audio-muted.png'

// Props & Refs
const props = defineProps({
  audioSrc: {
    type: String,
    required: true
  },
  isPlaying: {
    type: Boolean,
    required: true
  }
})

const audioPlayerContainer = ref<HTMLElement | null>(null)
const audioElement = ref<HTMLAudioElement | null>(null)
const progressContainer = ref<HTMLElement | null>(null)
const isMuted = ref(false)
const isLoading = ref(true)
const duration = ref(0)
const currentTime = ref(0)
const seekValue = ref(0)
const isDragging = ref(false)
const previewTime = ref(0)
const wasPlaying = ref(false)

const emit = defineEmits(['playStateChange', 'audioEnded'])

// 计算进度条样式
const progressBarStyle = computed(() => {
  const percentage = ((isDragging.value ? previewTime.value : currentTime.value) / duration.value) * 100
  return {
    backgroundSize: `${percentage}% 100%`
  }
})

// Methods
const calculateTime = (secs: number) => {
  const hours = Math.floor(secs / 3600)
  const minutes = Math.floor((secs % 3600) / 60)
  const seconds = Math.floor(secs % 60)
  const formattedMinutes = minutes < 10 ? `0${minutes}` : minutes
  const formattedseconds = seconds < 10 ? `0${seconds}` : seconds

  return hours > 0
    ? `${hours}:${formattedMinutes}:${formattedseconds}`
    : `${minutes}:${formattedseconds}`
}

const togglePlay = () => {
  emit('playStateChange', !props.isPlaying)
}

const toggleMute = () => {
  isMuted.value = !isMuted.value
}

// 计算点击或触摸位置对应的时间
const calculateTimeFromEvent = (clientX: number) => {
  if (!progressContainer.value || !audioElement.value) return 0
  
  const rect = progressContainer.value.getBoundingClientRect()
  const offsetX = clientX - rect.left
  const percentage = Math.max(0, Math.min(1, offsetX / rect.width))
  return percentage * duration.value
}

// 处理进度条点击
const onProgressBarClick = (event: MouseEvent) => {
  const newTime = calculateTimeFromEvent(event.clientX)
  if (audioElement.value) {
    audioElement.value.currentTime = newTime
    currentTime.value = newTime
    seekValue.value = newTime
  }
}

// 触摸事件处理
const onTouchStart = (event: TouchEvent) => {
  isDragging.value = true
  wasPlaying.value = props.isPlaying
  if (props.isPlaying) {
    emit('playStateChange', false)
  }
  onTouchMove(event)
}

const onTouchMove = (event: TouchEvent) => {
  if (!isDragging.value) return
  
  const touch = event.touches[0]
  previewTime.value = calculateTimeFromEvent(touch.clientX)
  seekValue.value = previewTime.value
}

const onTouchEnd = () => {
  if (!isDragging.value) return
  
  isDragging.value = false
  if (audioElement.value) {
    audioElement.value.currentTime = previewTime.value
    currentTime.value = previewTime.value
  }
  
  if (wasPlaying.value) {
    emit('playStateChange', true)
  }
}

// 开始拖动
const startDragging = (event: MouseEvent) => {
  isDragging.value = true
  wasPlaying.value = props.isPlaying
  if (props.isPlaying) {
    emit('playStateChange', false)
  }
  onDragging(event)
}

// 拖动中
const onDragging = (event: MouseEvent) => {
  if (!isDragging.value) return
  previewTime.value = calculateTimeFromEvent(event.clientX)
  seekValue.value = previewTime.value
}

// 停止拖动
const stopDragging = () => {
  if (!isDragging.value) return
  
  isDragging.value = false
  if (audioElement.value) {
    audioElement.value.currentTime = previewTime.value
    currentTime.value = previewTime.value
  }
  
  if (wasPlaying.value) {
    emit('playStateChange', true)
  }
}

// Computed
const currentTimeFormatted = computed(() => calculateTime(currentTime.value))
const durationFormatted = computed(() => calculateTime(duration.value))

// Watch
watch(() => props.isPlaying, (newValue) => {
  if (!audioElement.value) return

  if (newValue) {
    audioElement.value.play()
  } else {
    audioElement.value.pause()
  }
})

watch(isMuted, (newValue) => {
  if (audioElement.value) {
    audioElement.value.muted = newValue
  }
})

watch(() => props.audioSrc, (newSrc) => {
  if (newSrc) {
    isLoading.value = true
    duration.value = 0
    currentTime.value = 0
    seekValue.value = 0
    
    nextTick(() => {
      if (audioElement.value) {
        audioElement.value.src = newSrc
        audioElement.value.play()
          .then(() => {
            emit('playStateChange', true)
          })
          .catch((error) => {
            console.error('播放失败:', error)
            emit('playStateChange', false)
          })
      }
    })
  }
}, { immediate: true })

// Lifecycle
onMounted(() => {
  if (!audioElement.value) return

  const audio = audioElement.value

  audio.addEventListener('loadedmetadata', () => {
    duration.value = Math.floor(audio.duration)
    isLoading.value = false
  })

  audio.addEventListener('timeupdate', () => {
    if (!isDragging.value) {
      currentTime.value = Math.floor(audio.currentTime)
      seekValue.value = currentTime.value
    }
  })

  audio.addEventListener('ended', () => {
    emit('audioEnded')
  })

  // 添加键盘控制
  document.addEventListener('keydown', (e) => {
    if (e.code === 'Space') {
      e.preventDefault()
      togglePlay()
    }
  })
})
</script>

<style scoped>
input[type="range"] {
  height: 4px;
  background: #e5e7eb;
  border-radius: 8px;
  background-image: linear-gradient(#3b82f6, #3b82f6);
  background-size: 0% 100%;
  background-repeat: no-repeat;
  -webkit-appearance: none;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 12px;
  width: 12px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  box-shadow: 0 0 2px 0 #555;
  transition: background .3s ease-in-out;
}

input[type="range"]::-webkit-slider-thumb:hover {
  background: #2563eb;
}

input[type="range"]::-webkit-slider-runnable-track {
  -webkit-appearance: none;
  box-shadow: none;
  border: none;
  background: transparent;
}

/* 添加触摸设备的样式优化 */
@media (hover: none) {
  input[type="range"]::-webkit-slider-thumb {
    height: 16px;
    width: 16px;
  }
}
</style>