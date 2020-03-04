<template>
  <div class="container">
    <canvas width="200" height="200" id="c"></canvas>
    <p v-bind:class="isInspireActive ? 'isActive' : ''">I: {{inspire}}</p>
    <p v-bind:class="isHoldActive ? 'isActive' : ''">H: {{hold}}</p>
    <p v-bind:class="isExpireActive ? 'isActive' : ''">P: {{expire}}</p>
    <p>C: {{current}}</p>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

function getRadiusRatio({
  current, inspire, expire, hold,
}) {
  if (current <= inspire) return current / inspire;
  if (current <= inspire + hold) return 1;
  return 1 - (current - inspire - hold) / expire;
}

function getRadius({
  current, inspire, expire, hold,
}) {
  const base = 50;
  const amplitude = 40;
  const ratio = getRadiusRatio({
    current, inspire, expire, hold,
  });
  return base + amplitude * ratio;
}

const xCenter = 100;
const yCenter = 100;
const angleOffset = -(Math.PI / 2);
const TWO_PI = Math.PI * 2;

function getLingrad(vueCanvas, startAngle, endAngle, radius, color1, color2) {
  const xStart = xCenter + Math.cos(startAngle) * radius;
  const xEnd = xCenter + Math.cos(endAngle) * radius;
  const yStart = yCenter + Math.sin(startAngle) * radius;
  const yEnd = yCenter + Math.sin(endAngle) * radius;

  const lingrad = vueCanvas.createLinearGradient(xStart, yStart, xEnd, yEnd);
  lingrad.addColorStop(0, color1);
  lingrad.addColorStop(1, color2);

  return lingrad;
}

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

  getRadius() {
    const {
      current, inspire, expire, hold,
    } = this;
    return getRadius({
      current, inspire, expire, hold,
    });
  }

  getCycleLength() {
    return this.inspire + this.expire + this.hold;
  }

  drawBaseCircle() {
    this.vueCanvas.beginPath();
    const radius = this.getRadius();
    const x = 100;
    const y = 100;
    const startAngle = 0;
    const endAngle = TWO_PI;
    const anticlockwise = false;

    this.vueCanvas.arc(x, y, radius, startAngle, endAngle, anticlockwise);

    this.vueCanvas.lineWidth = 5;
    this.vueCanvas.lineCap = 'round';
    this.vueCanvas.fillStyle = 'white';
    this.vueCanvas.fill();
  }

  drawInspireArc() {
    const length = Math.min(this.current, this.inspire);
    const lengthRatio = length / this.inspire;
    const endRatio = this.inspire / this.getCycleLength();
    this.vueCanvas.beginPath();
    const radius = this.getRadius();
    const startAngle = angleOffset;
    const endAngle = angleOffset + (TWO_PI * endRatio * lengthRatio);
    const anticlockwise = false;

    this.vueCanvas.arc(xCenter, yCenter, radius, startAngle, endAngle, anticlockwise);
    this.vueCanvas.strokeStyle = getLingrad(this.vueCanvas, startAngle, endAngle, radius, '#00ABEB', '#26C000');
    this.vueCanvas.stroke();
  }

  drawHoldArc() {
    const length = Math.min(this.current - this.inspire, this.hold);
    if (length <= 0) return;
    const lengthRatio = length / this.hold;
    const endRatio = this.hold / this.getCycleLength();
    this.vueCanvas.beginPath();
    const radius = this.getRadius(); // Arc radius
    const holdAngleStart = (TWO_PI * (this.inspire / this.getCycleLength()));
    const startAngle = angleOffset + holdAngleStart;
    const endAngle = startAngle + (TWO_PI * endRatio * lengthRatio);
    const anticlockwise = false;

    this.vueCanvas.arc(xCenter, yCenter, radius, startAngle, endAngle, anticlockwise);
    this.vueCanvas.strokeStyle = getLingrad(this.vueCanvas, startAngle, endAngle, radius, '#F0F', '#F00');
    this.vueCanvas.stroke();
  }

  drawExpireArc() {
    const length = Math.min(this.current - this.inspire - this.hold, this.expire);
    if (length <= 0) return;
    const lengthRatio = length / this.expire;
    const endRatio = this.expire / this.getCycleLength();
    this.vueCanvas.beginPath();
    const radius = this.getRadius();
    const expireAngleStart = (TWO_PI * ((this.inspire + this.hold) / this.getCycleLength()));
    const startAngle = angleOffset + expireAngleStart;
    const endAngle = startAngle + (TWO_PI * endRatio * lengthRatio);
    const anticlockwise = false;

    this.vueCanvas.arc(xCenter, yCenter, radius, startAngle, endAngle, anticlockwise);
    this.vueCanvas.strokeStyle = getLingrad(this.vueCanvas, startAngle, endAngle, radius, '#FF0', '#F0F');
    this.vueCanvas.stroke();
  }

  renderWatch() {
    this.vueCanvas.clearRect(0, 0, 200, 200);
    this.drawBaseCircle();
    this.drawInspireArc();
    this.drawHoldArc();
    this.drawExpireArc();
  }

  mounted() {
    const c = document.getElementById('c');
    const ctx = c.getContext('2d');
    this.vueCanvas = ctx;
    this.renderWatch();
  }

  beforeUpdate() {
    this.renderWatch();
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
