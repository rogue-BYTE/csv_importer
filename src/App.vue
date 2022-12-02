<template>
  <div id="app">
    <CSVImportTrigger />
  </div>
</template>

<script>
import CSVImportTrigger from './components/CSVImportTrigger.vue'

export default {
  name: 'App',
  components: {
    CSVImportTrigger
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
