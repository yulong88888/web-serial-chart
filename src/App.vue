<template>
  <div id="app">
    <serial-chart></serial-chart>
  </div>
</template>

<script>

import SerialChart from "@/components/SerialChart";

export default {
  name: 'App',
  components: {
    SerialChart
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
</style>
