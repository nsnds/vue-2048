<template>
  <div id="app">
    <Index></Index>
  </div>
</template>

<script>
import Index from 'components/index'

export default {
  name: 'App',
  components: {
    Index
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body {
  background-color: #faf8ef;
}
</style>
