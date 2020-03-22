<template>
  <div class="container">
    <canvas id="c"></canvas>
    <div class="text-container">
      <p v-bind:class="isInspireActive ? 'isActive' : ''">Inspire: {{inspire}}</p>
      <p v-bind:class="isHoldActive ? 'isActive' : ''">Hold: {{hold}}</p>
      <p v-bind:class="isExpireActive ? 'isActive' : ''">Expire: {{expire}}</p>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

function getRadiusRatio({
  current, inspire, expire, hold,
}: {
  current: number;
  inspire: number;
  expire: number;
  hold: number;
}) {
  if (current <= inspire) return current / inspire;
  if (current <= inspire + hold) return 1;
  return 1 - (current - inspire - hold) / expire;
}

function getRadius({
  current, inspire, expire, hold,
}: {
  current: number;
  inspire: number;
  expire: number;
  hold: number;
}) {
  const base = 50;
  const amplitude = 40;
  const ratio = getRadiusRatio({
    current, inspire, expire, hold,
  });
  return base + amplitude * ratio;
}

const canvasSize = 300;
const xCenter = canvasSize / 2;
const yCenter = canvasSize / 2;
const angleOffset = -(Math.PI / 2);
const TWO_PI = Math.PI * 2;

function getLingrad(
  vueCanvas: CanvasRenderingContext2D,
  startAngle: number,
  endAngle: number,
  radius: number,
  color1: string,
  color2: string,
) {
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

  private vueCanvas: CanvasRenderingContext2D | null = null;

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

  getLineWidth() {
    return this.getRadius() / 8;
  }

  getCycleLength() {
    return this.inspire + this.expire + this.hold;
  }

  // eslint-disable-next-line class-methods-use-this
  getAngleInspireStart() {
    return angleOffset;
  }

  getAngleInspireCurrent() {
    const length = Math.min(this.current, this.inspire);
    const lengthRatio = length / this.inspire;
    const endRatio = this.inspire / this.getCycleLength();
    return angleOffset + (TWO_PI * endRatio * lengthRatio);
  }

  getAngleHoldStart() {
    return angleOffset + (TWO_PI * (this.inspire / this.getCycleLength()));
  }

  getAngleHoldCurrent() {
    const length = Math.min(this.current - this.inspire, this.hold);
    if (length <= 0) return angleOffset;
    const lengthRatio = length / this.hold;
    const endRatio = this.hold / this.getCycleLength();
    return this.getAngleHoldStart() + (TWO_PI * endRatio * lengthRatio);
  }

  getAngleExpireStart() {
    return angleOffset + (TWO_PI * ((this.inspire + this.hold) / this.getCycleLength()));
  }

  getAngleExpireCurrent() {
    const length = Math.min(this.current - this.inspire - this.hold, this.expire);
    if (length <= 0) return angleOffset;
    const lengthRatio = length / this.expire;
    const endRatio = this.expire / this.getCycleLength();
    return this.getAngleExpireStart() + (TWO_PI * endRatio * lengthRatio);
  }

  drawBaseCircle() {
    if (!this.vueCanvas) return;
    this.vueCanvas.beginPath();
    const radius = this.getRadius();
    const x = canvasSize / 2;
    const y = canvasSize / 2;
    const startAngle = 0;
    const endAngle = TWO_PI;
    const anticlockwise = false;

    this.vueCanvas.arc(x, y, radius, startAngle, endAngle, anticlockwise);

    this.vueCanvas.lineWidth = 5;
    this.vueCanvas.lineCap = 'round';
    this.vueCanvas.fillStyle = '#F0F0F0';
    this.vueCanvas.fill();
  }

  drawInspireArc() {
    if (!this.vueCanvas) return;
    this.vueCanvas.beginPath();
    const radius = this.getRadius();
    const startAngle = this.getAngleInspireStart();
    const endAngle = this.getAngleInspireCurrent();

    this.vueCanvas.arc(xCenter, yCenter, radius, startAngle, endAngle);
    this.vueCanvas.strokeStyle = getLingrad(this.vueCanvas, startAngle, endAngle, radius, '#00ABEB', '#00FFA9');
    this.vueCanvas.lineWidth = this.getLineWidth();
    this.vueCanvas.stroke();
  }

  drawHoldArc() {
    if (!this.vueCanvas) return;
    this.vueCanvas.beginPath();
    const startAngle = this.getAngleHoldStart();
    const endAngle = this.getAngleHoldCurrent();
    const radius = this.getRadius();
    if (endAngle === angleOffset) return;

    this.vueCanvas.arc(xCenter, yCenter, radius, startAngle, endAngle);
    this.vueCanvas.strokeStyle = getLingrad(this.vueCanvas, startAngle, endAngle, radius, '#FF0A6C', '#FF0061');
    this.vueCanvas.lineWidth = this.getLineWidth();
    this.vueCanvas.stroke();
  }

  drawExpireArc() {
    if (!this.vueCanvas) return;
    this.vueCanvas.beginPath();
    const startAngle = this.getAngleExpireStart();
    const endAngle = this.getAngleExpireCurrent();
    const radius = this.getRadius();
    if (endAngle === angleOffset) return;

    this.vueCanvas.arc(xCenter, yCenter, radius, startAngle, endAngle);
    this.vueCanvas.strokeStyle = getLingrad(this.vueCanvas, startAngle, endAngle, radius, '#A081FA', '#7BB9F6');
    this.vueCanvas.lineWidth = this.getLineWidth();
    this.vueCanvas.stroke();
  }

  drawADelimiter(angle: number) {
    if (!this.vueCanvas) return;
    this.vueCanvas.fillStyle = 'black';
    const delimiterRadius = this.getLineWidth();
    const baseRadius = this.getRadius();
    const x = xCenter + Math.cos(angle) * baseRadius;
    const y = yCenter + Math.sin(angle) * baseRadius;

    this.vueCanvas.beginPath();
    this.vueCanvas.arc(x, y, delimiterRadius, 0, TWO_PI);
    this.vueCanvas.fill();
    this.vueCanvas.closePath();
  }

  drawDelimiters() {
    if (!this.vueCanvas) return;
    this.drawADelimiter(this.getAngleInspireStart());
    this.drawADelimiter(this.getAngleHoldStart());
    this.drawADelimiter(this.getAngleExpireStart());
  }

  renderWatch() {
    if (!this.vueCanvas) return;
    this.vueCanvas.clearRect(0, 0, canvasSize, canvasSize);
    this.drawBaseCircle();
    this.drawInspireArc();
    this.drawHoldArc();
    this.drawExpireArc();
    this.drawDelimiters();
  }

  mounted() {
    const canvas = document.getElementById('c') as HTMLCanvasElement;
    this.vueCanvas = canvas.getContext('2d');

    // Set display canvasSize (css pixels).
    canvas.style.width = `${canvasSize}px`;
    canvas.style.height = `${canvasSize}px`;

    // Set actual canvasSize in memory (scaled to account for extra pixel density).
    // <--- Change to 1 on retina screens to see blurry canvas.
    const scale = window.devicePixelRatio;
    canvas.width = canvasSize * scale;
    canvas.height = canvasSize * scale;

    // Normalize coordinate system to use css pixels.
    if (this.vueCanvas) {
      this.vueCanvas.scale(scale, scale);
      this.vueCanvas.translate(0.5, 0.5); // avoid blurry renders
    }
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
.text-container {
  margin-top: 50px;
}
.isActive {
  font-weight: 800;
  color: #42b983;
}
</style>
