<template>
  <div id="app">
    <CssParser/>
  </div>
</template>

<script>
import CssParser from './components/CssParser.vue'

export default {
  name: 'App',
  components: {
    CssParser
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #c2c2c2d4;
  background: #101010e6;
  height: 100%;
  width: 100%;
}

html {
  background: #101010e6;
  height: 100%;
  width: 100%;
}

body {
  height: 100%;
  overflow-y: hidden;
}
</style>
