<template>
  <div class="container mx-auto lg:max-w-[960px] pb-[100px]">
    <div class="md:flex md:justify-center w-full">
      <div class="card min-w-full bg-[#fff] shadow-md  w-full">
        <div class="aspect-[3/1] w-full">
          <div class="relative w-full">
            <img src='./assets/banner.webp' class="object-cover w-full aspect-[3/1]" alt="郭德纲相声合集在线免费收听" />
            <div
              class="absolute top-0 left-0 items-center justify-center w-full h-full bg-gradient-to-t from-gray-900 to-50% to-transparent">
              <div class="flex flex-col absolute bottom-0 px-6">
                <h1 class="mb-3 text-lg md:text-4xl font-black text-white">郭德纲相声合集在线免费收听</h1>
                <p class="text-white text-sm md:text-base mb-4 max-w-2xl">收录郭德纲于谦最新相声，德云社相声全集，包括《论捧哏》《我要上春晚》《济公传》等经典相声作品，在线免费听郭德纲相声。</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="mt-2 py-2 min-w-full bg-[#fff] shadow-md px-6 rounded-lg">出售U盘内含本站所有音频:￥69，加微信：zxx960</div>
    <div class="mt-2 md:mt-2 min-w-full bg-[#fff] shadow-md px-6 rounded-lg">
      <div class="flex items-center py-3 border-b border-gray-200" v-for="(item, index) in audioList">
        <div class="flex-1 ml-5 flex items-center">
          <h2 class="md:text-xl text-sm font-semibold">
            <span class="text-gray-500">{{index+1}}. </span>
            <span>{{item.title}}</span>
          </h2>
          <div class="text-xs font-medium opacity-60 ml-2">{{item.time}}</div>
          <div class="my-1 text-xs md:text-sm line-clamp-2 text-gray-600">{{item.description}}</div>
        </div>
        <div class="ml-5">
          <div class="p-2 rounded-full bg-[#f0f0f0] cursor-pointer" @click="playAudio(item.url, index)">
            <img 
              :src="currentPlayingUrl === item.url && isPlaying ? PauseIcon : PlayIcon" 
              :alt="currentPlayingUrl === item.url && isPlaying ? '暂停播放郭德纲相声' : '播放郭德纲相声'" 
              class="w-6 h-6"
            >
          </div>
        </div>
      </div>
    </div>
    
    <!-- SEO 文本 -->
    <div class="mt-4 bg-white p-6 rounded-lg shadow-md">
      <h2 class="text-xl font-bold mb-4">关于郭德纲相声</h2>
      <div class="text-gray-600 text-sm space-y-4">
        <p>郭德纲，著名相声演员，德云社创始人。自1995年创办德云社以来，培养了众多优秀相声演员，为传统相声艺术的传承和发展做出了重要贡献。</p>
        <p>本站收录了郭德纲与搭档于谦的大量经典相声作品，包括《论捧哏》《我要上春晚》《济公传》《白蛇传》《学小曲》等。所有相声作品都可以在线免费收听，让您随时随地享受欢乐。</p>
        <p>郭德纲的相声特点是语言犀利，包袱密集，与搭档于谦配合默契。他的作品既保持了传统相声的艺术特色，又融入了现代元素，深受观众喜爱。</p>
      </div>
    </div>
  </div>
  <AudioPlayer 
    v-if="audioSrc"
    :audioSrc="audioSrc" 
    :isPlaying="isPlaying"
    @playStateChange="onPlayStateChange"
    @audioEnded="onAudioEnded"
  />
</template>
<script setup>
import { ref } from 'vue'
import PlayIcon from './assets/play.png'
import PauseIcon from './assets/pause.png'
import AudioPlayer from './components/AudioPlayer.vue'
import { audioList } from '../config/audio-list.json'

const audioSrc = ref('')
const currentPlayingUrl = ref('')
const isPlaying = ref(false)
const currentIndex = ref(-1)

function playAudio(url, index) {
  if (currentPlayingUrl.value === url) {
    // 如果点击的是当前正在播放的音频，则切换播放状态
    isPlaying.value = !isPlaying.value
  } else {
    // 如果是新的音频，开始播放
    currentPlayingUrl.value = url
    currentIndex.value = index
    audioSrc.value = url
    isPlaying.value = true
  }
}

// 播放下一个音频
function playNext() {
  if (currentIndex.value === -1) return
  const nextIndex = (currentIndex.value + 1) % audioList.length
  const nextAudio = audioList[nextIndex]
  playAudio(nextAudio.url, nextIndex)
}

// 监听播放器的播放状态
const onPlayStateChange = (playing) => {
  isPlaying.value = playing
}

// 监听音频播放结束
const onAudioEnded = () => {
  playNext()
}
</script>
<style scoped>
.card {
  position: relative;
  display: flex;
  flex-direction: column;
  border-radius: var(--rounded-box, 1rem);
}
</style>
