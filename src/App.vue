<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
  export default {
    name: 'App'
  }
</script>

<style>
  #app {
    margin: 20px;
  }

  * {
    box-sizing: border-box;
  }

  body {
    margin: 0 auto;
    padding: 0;
  }
</style>
