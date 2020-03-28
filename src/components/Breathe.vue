<template>
  <div class="background">
    <div v-on:click="toggle">
      <Watch
        :inspire="inspire"
        :hold="hold"
        :expire="expire"
        :current="current"
      />
    </div>
    <div class="bottom-row">
      <div class="nb-cycles-container">
        <p class="nb-cycles">{{nbCycles}}</p>
      </div>
      <div
        v-on:click="toggleSound"
        v-bind:class="isSoundActive ? 'icon-sound icon-sound_active' : 'icon-sound'"
      >
        <ion-icon
          v-bind:name="isSoundActive ? 'volume-high-outline' : 'volume-off-outline'"
        ></ion-icon>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { Howl } from 'howler';
import Watch from './Watch.vue';

const INCREMENT_DURATION_SECONDS = 0.01;

@Component({
  components: {
    Watch,
  },
})
export default class Breathe extends Vue {
  current = 0

  nbCycles = 0

  inspire = 4

  hold = 4

  expire = 4

  timeout: number | null = null

  isSoundActive = false

  soundGong: Howl = new Howl({ src: ['gong.mp3'] })

  getCycleLength() {
    return this.inspire + this.expire + this.hold;
  }

  shouldPlaySound() {
    if (!this.isSoundActive) return false;
    const inspireStart = 0;
    const holdStart = this.inspire;
    const expireStart = this.inspire + this.hold;
    const starts = [inspireStart, holdStart, expireStart];
    return starts.includes(this.current);
  }

  increment() {
    this.current = Math.round((this.current + INCREMENT_DURATION_SECONDS) * 100) / 100;
    if (this.current >= this.getCycleLength()) {
      this.current = 0;
      this.nbCycles += 1;
    }
    if (this.shouldPlaySound()) {
      if (this.soundGong) {
        this.soundGong.play();
      }
    }
    this.timeout = setTimeout(() => this.increment(), INCREMENT_DURATION_SECONDS * 1000);
  }

  toggle() {
    if (this.timeout) {
      clearTimeout(this.timeout);
      this.timeout = null;
    } else {
      if (this.soundGong && this.isSoundActive) {
        this.soundGong.play();
      }
      this.increment();
    }
  }

  toggleSound() {
    this.isSoundActive = !this.isSoundActive;
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

.bottom-row {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  width: calc(100vw - 20px);
  position: fixed;
  bottom: 0px;
  margin: 16px 10px;
}

.nb-cycles-container {
  margin: 0px;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 32px;
  width: 32px;
}

.nb-cycles {
  color: white;
  margin: 8px 0px 0px;
}

.icon-sound {
  cursor: pointer;
  font-size: 32px;
  height: 32px;
  transition: all 0.3s ease;
}

.icon-sound_active {
  font-size: 32px;
  height: 32px;
  color: white;
  right: 8px;
  bottom: 8px;
}
</style>
