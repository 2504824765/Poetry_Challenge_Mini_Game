<template>
  <div id="app">
    <GameCanvas />
  </div>
</template>

<script>
import GameCanvas from './components/GameCanvas.vue'

export default {
  name: 'App',
  components: {
    GameCanvas
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
  background-color: #f5f5f5;
}
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
</style>