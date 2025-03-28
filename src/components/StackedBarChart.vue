<template>
  <div ref="chart"></div>
</template>

<script>
import * as d3 from 'd3';

export default {
  props: ['data'],
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      const data = this.data;
      const margin = { top: 20, right: 20, bottom: 30, left: 40 };
      const width = 960 - margin.left - margin.right;
      const height = 500 - margin.top - margin.bottom;

      const svg = d3.select(this.$refs.chart)
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = d3.scaleLinear()
        .range([0, width]);

      const y = d3.scaleBand()
        .range([height, 0])
        .padding(0.1);

      const z = d3.scaleOrdinal()
        .range(['#98abc5', '#8a89a6', '#7b6888', '#6b486b', '#a05d56', '#d0743c', '#ff8c00']);

      const keys = Object.keys(data[0]).slice(1);

      x.domain([0, d3.max(data, d => d3.sum(keys, key => d[key]))]).nice();
      y.domain(data.map(d => d.name));
      z.domain(keys);

      svg.append('g')
        .selectAll('g')
        .data(d3.stack().keys(keys)(data))
        .enter().append('g')
          .attr('fill', d => z(d.key))
        .selectAll('rect')
        .data(d => d)
        .enter().append('rect')
          .attr('y', d => y(d.data.name))
          .attr('x', d => x(d[0]))
          .attr('width', d => x(d[1]) - x(d[0]))
          .attr('height', y.bandwidth());

      // Add the X Axis
      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(d3.axisBottom(x).ticks(null, 's'));

      // Add the Y Axis
      svg.append('g')
        .call(d3.axisLeft(y));
    }
  }
};
</script>