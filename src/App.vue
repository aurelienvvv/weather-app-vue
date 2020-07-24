<template>
  <div id="app">
    <meteo></meteo>
  </div>
</template>

<script>
import Meteo from "./components/Meteo.vue";

export default {
  name: "App",
  components: {
    Meteo
  }
};
</script>

<style lang='scss'>
@import "App.scss";
</style>
