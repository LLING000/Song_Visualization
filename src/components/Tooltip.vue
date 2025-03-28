<template>
    <div :style="tooltipStyle" class="tooltip">
      <div class="tooltip-content">
        <h2>{{ title }}</h2>
        <p v-html="content"></p>
        <!-- <div v-for="book in matchingBooks" :key="book">{{ book }}</div> -->
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      x: Number,
      y: Number,
      info: String,
      charData: Array
    },
    computed: {
      tooltipStyle() {
        return {
          position: 'absolute',
          left: `${this.x}px`,
          top: `${this.y}px`,
          backgroundColor: '#F8E6E0',
          color: '#5D4037',
          padding: '15px',
          borderRadius: '10px',
          pointerEvents: 'none',
          fontSize: '14px',
          zIndex: 10,
          width: '150px',
          boxShadow: '0 4px 8px rgba(0, 0, 0, 0.1)',
          whiteSpace: 'pre-line'  // 保留换行
        };
      },
      title() {
        // 假设标题在 info 的第一行
        return this.info.split('\n')[0];
      },
      content() {
        // 假设内容是 info 的其余部分
        return this.info.split('\n').slice(1).join('<br>');
      }
    }
  };
  </script>
  
  <style scoped>
  .tooltip {
    display: inline-block;
    overflow: auto;
  }
  .chart::-webkit-scrollbar {
    width: 0; /* 设置滚动条的宽度为 0，使其不可见 */
  }
  .tooltip-content {
    font-family: '宋体', sans-serif;
  }
  .tooltip-content h2 {
    font-size: 18px;
    margin-bottom: 10px;
    border-bottom: 1px solid #5D4037;
  }
  .tooltip-content p {
    font-size: 14px;
    line-height: 1.5;
  }
  </style>
  