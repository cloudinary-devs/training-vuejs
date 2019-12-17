<template>
  <div id="app">
    <img class="vue-logo" src="./assets/logo.png" />
    <cl-upload class="align" />
  </div>
</template>

<script>
import CloudinaryUpload from "./components/CloudinaryUpload";

export default {
  name: "App",
  components: {
    "cl-upload": CloudinaryUpload
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

body {
  display: grid;
  place-items: center;
  height: 80vh;
}
.vue-logo {
  width: 50px;
  height: auto;
}
</style>
