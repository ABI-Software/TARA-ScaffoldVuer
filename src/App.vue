<script setup>
import TaraScaffoldVuer from './components/TaraScaffoldVuer.vue';
</script>

<template>
  <div id="app">
    <TaraScaffoldVuer :url="url" />
  </div>
</template>

<script>
export default {
  name: "app",
  data: function () {
    return {
      url: "https://mapcore-bucket1.s3.us-west-2.amazonaws.com/texture/arm1/arm_metadata.json"
    }
  }
}

</script>

<style>
#app {
  font-family: "Asap", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
  width: 100%;
  position: absolute;
  overflow: hidden;
}

body {
  margin: 0px;
  overflow: hidden;
}
/* Component Styles */
</style>
