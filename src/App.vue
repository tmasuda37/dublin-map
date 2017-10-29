<template>
  <div id="app">
    <GoogleMaps/>
  </div>
</template>

<script>
import GoogleMaps from './components/GoogleMaps'

export default {
  name: 'app',
  components: {
    GoogleMaps
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
