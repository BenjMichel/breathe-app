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
import { Howl } from 'howler';
import Watch from './Watch.vue';

@Component({
  components: {
    Watch,
  },
})
export default class Breathe extends Vue {
  current = 0

  inspire = 4

  hold = 4

  expire = 4

  timeout: number | null = null

  soundGong: Howl = new Howl({ src: ['gong.mp3'] })

  getCycleLength() {
    return this.inspire + this.expire + this.hold;
  }

  shouldPlaySound() {
    const inspireStart = 0;
    const holdStart = this.inspire;
    const expireStart = this.inspire + this.hold;
    const starts = [inspireStart, holdStart, expireStart];
    return starts.includes(this.current);
  }

  increment() {
    this.current = Math.round((this.current + 0.1) * 10) / 10;
    if (this.current >= this.getCycleLength()) this.current = 0;
    if (this.shouldPlaySound()) {
      if (this.soundGong) {
        this.soundGong.play();
      }
    }
    this.timeout = setTimeout(() => this.increment(), 100);
  }

  toggle() {
    if (this.timeout) {
      clearTimeout(this.timeout);
      this.timeout = null;
    } else {
      if (this.soundGong) {
        this.soundGong.play();
      }
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
