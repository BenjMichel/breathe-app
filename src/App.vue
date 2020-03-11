<template>
  <div id="app">
    <Breathe msg="Welcome to Your Vue.js + TypeScript App"/>
    <link href="https://fonts.googleapis.com/css?family=Spartan&display=swap" rel="stylesheet">
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Breathe from './components/Breathe.vue';

@Component({
  components: {
    Breathe,
  },
})
export default class App extends Vue {}
</script>

<style>
#app {
  font-family: Spartan, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0px;
}

::-moz-selection { color: transparent}
::selection { color: transparent}

body,html {
  margin: 0px;
}
</style>
