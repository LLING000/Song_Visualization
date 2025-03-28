<template>
  <div class="bar-chart">
    <svg :width="width" :height="height" :viewBox="viewBox">
      <g v-for="(bar, index) in bars" :key="index" :transform="'translate(0,' + index * barHeight + ')'">
        <rect v-for="(segment, i) in bar.segments"
              :key="i"
              :x="calculateX(segment, i, bar.segments)"
              :y="0"
              :width="segment.width"
              :height="barHeight - 1"
              :fill="segment.color"/>
        <text :x="calculateTextX(bar.segments)" :y="barHeight / 2" :dy=".35" class="label">{{ bar.label }}</text>
      </g>
    </svg>
  </div>
</template>

<script>
export default {
  name: 'BarChart',
  props: {
    bars: {
      type: Array,
      required: true
    },
    width: {
      type: Number,
      default: 300
    },
    height: {
      type: Number,
      default: 600
    },
    margin: {
      type: Number,
      default: 0
    },
    spacing: {
      type: Number,
      default: 40 // 默认间距，可以根据需要调整
    }
  },
  computed: {
    barHeight() {
      return this.height / this.bars.length;
    },
    viewBox() {
      return `-${this.margin +15} 0 ${this.width + this.margin} ${this.height+30}`;
    }
  },
  methods: {
    calculateX(segment, index, segments) {
      return segments.slice(0, index).reduce((acc, seg) => acc + seg.width, 0);
    },
    calculateTextX(segments) {
      return -10; // 调整文本标签的位置，使其在柱子的左边
    }
  }
}
</script>

<style scoped>
.label {
  fill: white; /* 文本标签的颜色 */
  font-size: 8px; /* 可以根据需要调整字体大小 */
  text-anchor: end; /* 文本对齐到x坐标的末端 */
}
</style>
