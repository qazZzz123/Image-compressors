<script lang="ts" setup>
import ImageCompressor from './components/ImageCompressor.vue'
</script>

<template>
  <div class="app">
    <div class="background-animation"></div>
    <ImageCompressor />
  </div>
</template>

<style>
.app {
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  position: relative;
  overflow: hidden;
}

.background-animation {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 20% 20%, rgba(62, 184, 229, 0.1) 0%, transparent 50%),
    radial-gradient(circle at 80% 80%, rgba(183, 235, 143, 0.15) 0%, transparent 50%);
  animation: backgroundMove 15s ease-in-out infinite;
  z-index: 0;
}

@keyframes backgroundMove {
  0%, 100% {
    transform: translate(0, 0);
  }
  25% {
    transform: translate(2%, 2%);
  }
  50% {
    transform: translate(-1%, 1%);
  }
  75% {
    transform: translate(1%, -1%);
  }
}
</style>
