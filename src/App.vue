<template>
  <div id="app">
    <Budget />
  </div>
</template>

<script>
import Budget from './components/Budget.vue'

export default {
  name: 'App',
  components: {
    Budget
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
