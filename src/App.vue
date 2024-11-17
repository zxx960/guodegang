<template>
  <div class="container mx-auto lg:max-w-[960px] pb-[100px]">
    <div class="md:flex md:justify-center w-full">
      <div class="card min-w-full bg-[#fff] shadow-md  w-full">
        <div class="aspect-[3/1] w-full">
          <div class="relative w-full">
            <img src='./assets/banner.webp' class="object-cover w-full aspect-[3/1]" />
            <div
              class="absolute top-0 left-0 items-center justify-center w-full h-full bg-gradient-to-t from-gray-900 to-50% to-transparent">
              <div class="flex flex-col absolute bottom-0  px-6">
                <h1 class="mb-3 text-lg md:text-4xl font-black text-white">天涯神贴播客</h1>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="mt-2 py-2 min-w-full bg-[#fff] shadow-md px-6 rounded-lg">将本播客打包下载到本地收听，加微信：zxx960，长期更新（网盘版:￥99，u盘版:￥149）</div>
    <div class="mt-2 md:mt-2 min-w-full bg-[#fff] shadow-md px-6 rounded-lg">
      <div class="flex items-center py-6 border-b border-gray-200" v-for="item in audioList">
        <img src="./assets/cover.jpg" alt="天涯神贴播客" class="shadow-lg rounded-md w-[4.5rem] h-[4.5rem] md:w-24 md:h-24  ">
        <div class="flex-1 ml-5">
          <h2 class="md:text-xl text-sm font-semibold ">{{item.name}}</h2>
          <div class="text-xs font-medium opacity-60">{{item.time}}</div>
          <div class="my-1 text-xs md:text-sm line-clamp-2">{{item.description}}</div>
        </div>
        <div class="ml-5">
          <div class="p-2 rounded-full bg-[#f0f0f0] cursor-pointer" @click="playAudio(item.url)">
            <img 
              :src="currentPlayingUrl === item.url && isPlaying ? PauseIcon : PlayIcon" 
              :alt="currentPlayingUrl === item.url && isPlaying ? '暂停' : '播放'" 
              class="w-6 h-6"
            >
          </div>
        </div>
      </div>
    </div>
  </div>
  <AudioPlayer 
    v-if="audioSrc"
    :audioSrc="audioSrc" 
    :isPlaying="isPlaying"
    @playStateChange="onPlayStateChange"
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

function playAudio(url) {
  if (currentPlayingUrl.value === url) {
    // 如果点击的是当前正在播放的音频，则切换播放状态
    isPlaying.value = !isPlaying.value
  } else {
    // 如果是新的音频，开始播放
    currentPlayingUrl.value = url
    audioSrc.value = url
    isPlaying.value = true
  }
}

// 监听播放器的播放状态
const onPlayStateChange = (playing) => {
  isPlaying.value = playing
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
