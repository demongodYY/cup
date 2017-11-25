<template>
  <div id="app">
    <cup />
  </div>
</template>

<script>
import cup from './components/cup'
// import preview from './components/preview'

export default {
  name: 'app',
  components: {
    cup
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
</style>
