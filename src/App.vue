<script setup>
import TaraScaffoldVuer from './components/TaraScaffoldVuer.vue';
</script>

<template>
  <div id="app">
    <TaraScaffoldVuer
      :acupointsViewer="acupointsViewer"
      :url="url"
      :acupoints-endpoint="acupoints"
      :texture-url="textureURL"
      :console-on="false"
    />
  </div>
</template>

<script>
export default {
  name: "app",
  data: function () {
    return {
      /* Settings for standard viewer */
      acupointsViewer: false,
      url: "https://mapcore-bucket1.s3.us-west-2.amazonaws.com/texture/arm1/arm_metadata.json",
      /* Settings for acupoint viewer */
      //acupointsViewer: true,
      //url: "https://mapcore-bucket1.s3.us-west-2.amazonaws.com/tara/whole_body-30-1-25/human_body_acupoints_metadata.json",
      //textureURL: import.meta.env.VITE_TEXTURE_LOCATION,
      //acupoints: import.meta.env.VITE_ACUPOINTS_API,
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
