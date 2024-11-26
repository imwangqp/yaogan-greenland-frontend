<script>
import axios from "axios";
import _ from "lodash"
import 'proj4leaflet'
import * as d3 from 'd3'
import proj4 from "proj4"
import {wgs84togcj02, gcj02tobd09} from 'coordtransform'
import * as L from "leaflet"
import 'leaflet.chinatmsproviders'
import {EPSG4326} from "leaflet/src/geo/crs/CRS.EPSG4326.js";

import {ColorList} from "../util/global.js";

export default {
  name: "LeafletComponent",
  mounted() {
    // axios.get('api/coordinates_in_date', {
    //   params: {Date: '2019-11-24'}
    // }).then(res => {
    //   console.log(res)
    //   this.initMap()
    //   this.polygons = res.data
    //   this.initSvg(this.polygons);
    // })
    this.initMap()
    this.loadPolygons().then((polygons) => {
      this.polygons = polygons
      this.initSvg(this.polygons);
    });

  },
  data() {
    return {
      map: null,
      baseX: null,
      baseY: null,
      polygons: null
    }
  },
  methods: {
    initMap() {
      const map = L.map('map-container', {
        minZoom: 3,
        maxZoom: 18,
        center: [22.5445741, 114.0545429],
        zoom: 13,
        zoomControl: false,
        // crs: EPSG4326,
        attributionControl: false
      })
      this.map = map
      L.tileLayer.chinaProvider('TianDiTu.Normal.Map', {maxZoom: 18, minZoom: 5}).addTo(map);
    },
    initSvg(polygons) {



      let that = this

      const width = document.getElementById('svg-container').offsetWidth,
          height = document.getElementById('svg-container').offsetHeight

      const svg = d3.select('#svg-container')
          .append('svg')
          .attr('width', document.getElementById('svg-container').offsetWidth)
          .attr('height', document.getElementById('svg-container').offsetHeight)
          .on('wheel', (event) => {
            event.preventDefault();
            if (event.deltaY < 0) {
              this.map.zoomIn();
            } else if (event.deltaY > 0) {
              this.map.zoomOut();
            }
          });

      // 坐标转换函数
      const projectPoint = ([lng, lat]) => {
        const point = this.map.latLngToContainerPoint([lat, lng]); // 转换为像素坐标
        return [point.x, point.y];
      };

      const drag = d3.drag()
          .on('drag', (event) => {
            const dx = event.dx;
            const dy = event.dy;
            const currentCenter = this.map.getCenter();
            const newCenter = this.map.containerPointToLatLng(
                this.map.latLngToContainerPoint(currentCenter).subtract([dx, dy])
            );
            this.map.setView(newCenter, this.map.getZoom(), { animate: false });
            drawContour(); // 重新绘制多边形
          });

      svg.call(drag); // 将拖动绑定到 SVG 容器

      // 监听地图缩放和拖动事件
      this.map.on('zoom', drawContour);
      this.map.on('move', drawContour);

      function drawContour(){
        svg.selectAll('.contour').remove()
        const data = polygons.flatMap(polygon =>
            polygon.map(([lng, lat]) => {
              const point = that.map.latLngToContainerPoint([lat, lng]);
              return {
                x: point.x,
                y: point.y,
                value: Math.random() * 100 // 这里值可以是根据实际数据计算的
              };
            })
        );
        console.log(data)

        const contour = d3.contourDensity()
            .x(d => d.x)
            .y(d => d.y)
            .size([width, height])
            .thresholds(10)(data); // 设置等高线的阈值

        const contourColorScale = d3.scaleSequential(['white', ColorList.contourBgColor])
            .domain([
              d3.min(contour, (d) => d.value) * 0.9,
              d3.max(contour, (d) => d.value) * 1.1
            ])

        svg.selectAll('.contour')
            .data(contour)
            .enter().append('path')
            .attr('class', 'contour')
            .attr('d', d3.geoPath())
            .attr('fill', d => contourColorScale(d.value))
            .attr('opacity', 0.5);
      }

      drawContour()
    },
    async loadPolygons() {
      const response = await fetch('/output_polygon_vertices(1).txt');
      const text = await response.text();
      const polygons = [];
      let currentPolygon = [];
      text.split('\n').forEach(line => {
        if (line.startsWith('Polygon')) {
          if (currentPolygon.length > 0) {
            polygons.push(currentPolygon);
            currentPolygon = [];
          }
        } else if (line.trim().startsWith('[')) {
          const coords = line.trim().slice(1, -1).split(',').map(Number);
          currentPolygon.push(coords);
        }
      });
      if (currentPolygon.length > 0) {
        polygons.push(currentPolygon);
      }
      return polygons;
      // const [baseX, baseY] = polygons[0][0];
      // this.baseX = baseX;
      // this.baseY = baseY;
      //
      // // 计算所有顶点相对基准点的差值
      // return polygons.map(polygon =>
      //     polygon.map(([x, y]) => [(x - baseX) * 1000 + 400, -(y - baseY) * 1000 + 300]),
      // );
    }
  },


}
</script>

<template>
  <div class="full-container">
    <div class="child-container" id="map-container"></div>
    <div class="child-container" id="svg-container"></div>
  </div>
</template>

<style scoped>
.child-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.full-container {
  position: relative;
  width: 100%;
  height: 100%;
}

#map-container {
  position: absolute; /* 或者 relative，具体取决于你的布局需求 */
  z-index: 1; /* 设置较小的 z-index 让它位于下面 */
}

#svg-container {
  position: absolute; /* 或者 relative */
  z-index: 2; /* 设置较大的 z-index 让它位于上面 */
}
</style>