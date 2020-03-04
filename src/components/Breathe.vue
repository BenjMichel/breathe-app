<template>
  <div class="background" v-on:click="toggle">
    <Watch
      :inspire="inspire"
      :hold="hold"
      :expire="expire"
      :current="current"
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Watch from './Watch.vue';

@Component({
  components: {
    Watch,
  },
})
export default class Breathe extends Vue {
  current = 0

  inspire = 4

  hold = 2

  expire = 3

  timeout: number | null = null

  getCycleLength() {
    return this.inspire + this.expire + this.hold;
  }

  increment() {
    this.current = Math.round((this.current + 0.1) * 10) / 10;
    if (this.current >= this.getCycleLength()) this.current = 0;
    this.timeout = setTimeout(() => this.increment(), 100);
  }

  toggle() {
    if (this.timeout) {
      clearTimeout(this.timeout);
      this.timeout = null;
    } else {
      this.increment();
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.background {
  height: 100vh;
  width: 100vw;
  background-image: url("../assets/background.jpg");
  background-size: cover;
}
</style>
