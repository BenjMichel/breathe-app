<template>
  <div class="container">
    <p v-bind:class="isInspireActive ? 'isActive' : ''">I: {{inspire}}</p>
    <p v-bind:class="isHoldActive ? 'isActive' : ''">H: {{hold}}</p>
    <p v-bind:class="isExpireActive ? 'isActive' : ''">P: {{expire}}</p>
    <p>C: {{current}}</p>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class Breathe extends Vue {
  @Prop() private inspire!: number;

  @Prop() private hold!: number;

  @Prop() private expire!: number;

  @Prop() private current!: number;

  get isInspireActive() {
    return this.current < this.inspire;
  }

  get isHoldActive() {
    return this.current >= this.inspire && this.current < this.inspire + this.hold;
  }

  get isExpireActive() {
    return this.current >= this.inspire + this.hold;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: white;
}
.isActive {
  font-weight: 800;
  color: #42b983;
}
</style>
