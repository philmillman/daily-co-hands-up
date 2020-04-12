<template>
  <div id="app">
    <Call />
  </div>
</template>

<script>
import Call from './components/Call.vue'

export default {
  name: 'App',
  components: {
    Call
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
