<script>
import * as d3 from 'd3'
export default {
  name: "TimeController",
  mounted() {
    this.generateExampleData()
    this.initController()
  },
  data(){
    return {
      width:100,
      height:40,
      exampleData:[]
    }
  },
  methods: {
    generateExampleData(){
      const startDate = new Date();
      startDate.setDate(startDate.getDate() - 15);

      const maxValue = 100;

      this.exampleData = Array.from({ length: 15 }, (_, i) => {
        const time = new Date(startDate);
        time.setDate(time.getDate() + i); // 每天增加一天
        const value = Math.floor(Math.random() * maxValue); // 随机生成0到100的值
        return { time, value };
      });

      console.log(this.exampleData);

    },
    initController(){
      d3.select('#time-controller').selectAll('*').remove()
      const svg = d3.select('#time-controller')
          .append('svg')
          .attr('width', this.width)
          .attr('height', this.height)

      const xScale = d3.scaleTime()
          .domain(d3.extent(this.exampleData, d => new Date(d.time)))
          .range([0, this.width])

      const yScale = d3.scaleLinear()
          .domain([0, d3.max(this.exampleData, d=>d.value)])
          .range([this.height, 0])

      const areaGroup = svg.append('g')

      const defs = svg.append("defs");

      const gradient = defs.append("linearGradient")
          .attr("id", "blueToWhiteGradient")
          .attr("x1", "0%")
          .attr("y1", "0%")
          .attr("x2", "0%")
          .attr("y2", "100%")

      const area = d3.area()
          .x(d => xScale(d.time))
          .y0(yScale(0))
          .y1(d => yScale(d.value))

      gradient.append("stop")
          .attr("offset", "0%")
          .attr("style", "stop-color:steelblue; stop-opacity:1");

      gradient.append("stop")
          .attr("offset", "100%")
          .attr("style", "stop-color:white; stop-opacity:1");

      areaGroup.append('path')
          .attr('fill', "steelblue")
          .attr('d', area(this.exampleData))

      const brush = d3.brushX() // 创建刷子对象
          .extent([[0, 0], [this.width, this.height]]) // 刷子的活动区域
          .on("start brush end", brushed);

      areaGroup.call(brush)

      function brushed(e) {
        console.log(e)
      }
    }
  }
}
</script>

<template>
  <div id="time-controller"></div>
</template>

<style scoped>

</style>