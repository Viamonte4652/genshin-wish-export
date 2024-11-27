<template>
  <div class="titlebar bg-gray-900 flex justify-between items-center px-2 select-none">
    <div class="titlebar-drag flex-1 h-8"></div>
    <div class="window-controls flex">
      <button @click="minimize" class="h-8 w-12 hover:bg-gray-700 flex items-center justify-center text-gray-400 hover:text-white">
        <svg width="10" height="1" viewBox="0 0 10 1">
          <path d="M0 0h10v1H0z" fill="currentColor"/>
        </svg>
      </button>
      <button @click="maximize" class="h-8 w-12 hover:bg-gray-700 flex items-center justify-center text-gray-400 hover:text-white">
        <svg v-if="!isMaximized" width="10" height="10" viewBox="0 0 10 10">
          <path d="M0 0v10h10V0H0zm1 1h8v8H1V1z" fill="currentColor"/>
        </svg>
        <svg v-else width="10" height="10" viewBox="0 0 10 10">
          <path d="M2.1,0v2H0v8.1h8.2v-2h2V0H2.1z M7.2,9.2H1.1V3h6.1V9.2z M9.2,7.1h-1V2H3.1V1h6.1V7.1z" fill="currentColor"/>
        </svg>
      </button>
      <button @click="close" class="h-8 w-12 hover:bg-red-600 flex items-center justify-center text-gray-400 hover:text-white">
        <svg width="10" height="10" viewBox="0 0 10 10">
          <path d="M6.4,5l3.3-3.3c0.4-0.4,0.4-1,0-1.4s-1-0.4-1.4,0L5,3.6L1.7,0.3c-0.4-0.4-1-0.4-1.4,0s-0.4,1,0,1.4L3.6,5L0.3,8.3 c-0.4,0.4-0.4,1,0,1.4C0.5,9.9,0.7,10,1,10s0.5-0.1,0.7-0.3L5,6.4l3.3,3.3C8.5,9.9,8.7,10,9,10s0.5-0.1,0.7-0.3 c0.4-0.4,0.4-1,0-1.4L6.4,5z" fill="currentColor"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
const { ipcRenderer } = require('electron')
import { ref, onMounted } from 'vue'

const isMaximized = ref(false)

const minimize = () => {
  ipcRenderer.send('window-minimize')
}

const maximize = () => {
  ipcRenderer.send('window-maximize')
  isMaximized.value = !isMaximized.value
}

const close = () => {
  ipcRenderer.send('window-close')
}

onMounted(() => {
  ipcRenderer.on('window-maximized', () => {
    isMaximized.value = true
  })
  ipcRenderer.on('window-unmaximized', () => {
    isMaximized.value = false
  })
})
</script>

<style>
.titlebar {
  -webkit-app-region: drag;
  height: 32px;
}

.window-controls button {
  -webkit-app-region: no-drag;
}
</style>
