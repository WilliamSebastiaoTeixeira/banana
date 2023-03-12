<template>
  <BananaCanvas/>
</template>

<script setup>
  import { onMounted } from 'vue'
  import BananaCanvas from './components/BananaCanvas.vue'

  onMounted(() => document.title = 'Banana')
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  width: 100vw;
  height: 100vh;
}
</style>
