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

export default {
  name: "GreenLandComponent",
  mounted() {
    axios.get('api/coordinates_in_date', {
      params: {Date: '2019-11-24'}
    }).then(res => {
      console.log(res)
      this.initMap()
      this.polygons  = _.map(res.data, item => item['Coordinates'])
      console.log(this.polygons)
      this.initSvg(this.polygons);
    })
    // this.initMap()
    // this.loadPolygons().then((polygons) => {
    //   console.log(polygons)
    //   this.polygons = polygons
    //   this.initSvg(this.polygons);
    // });

  },
  data() {
    return {
      showPolygon: true,
      map: null,
      baseX: null,
      baseY: null,
      polygons: null
    }
  },
  methods: {
    // showPolygon(){
    //   console.log(123)
    // },
    initMap() {
      const map = L.map('map-container-2', {
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

      // axios({
      //   url: '/api/coordinates',
      //   params: {
      //     Date: '2020-01-08'
      //   }
      // }).then(res => {
      //   console.log(res)
      //   const map = L.map('map-container', {
      //     minZoom: 3,
      //     maxZoom: 18,
      //     center: [22.5445741, 114.0545429],
      //     zoom: 13,
      //     crs: L.CRS.Baidu
      //   })
      //   L.tileLayer.baidu({layer: 'vec'}).addTo(map)
      //   for (let i = 0; i < res.data.length; i++) {
      //     console.log(res.data[i]['Coordinates'])
      //     const transformedCoordinates = res.data[i]['Coordinates'].map(coord => {
      //       let [gcjLon, gcjLat] = wgs84togcj02(coord[0], coord[1]);
      //       return gcj02tobd09(gcjLon, gcjLat);
      //     });
      //     const polygon = L.polygon(_.map(transformedCoordinates, i=>[i[1], i[0]])).addTo(map)
      //   }
      // })
    },
    initSvg(polygons) {
      const svg = d3.select('#svg-container-2')
          .append('svg')
          .attr('width', document.getElementById('svg-container-2').offsetWidth)
          .attr('height', document.getElementById('svg-container-2').offsetHeight)
          .on('wheel', (event) => {
            event.preventDefault(); // 阻止页面滚动

            if (event.deltaY < 0) {
              this.map.zoomIn();
            } else if (event.deltaY > 0) {
              this.map.zoomOut();
            }
          });
      // 坐标转换函数
      const projectPoint = ([lat, lng]) => {
        const point = this.map.latLngToContainerPoint([lat, lng]); // 转换为像素坐标
        return [point.x, point.y];
      };

      // 绘制多边形
      const drawPolygons = () => {
        svg.selectAll('path').remove(); // 清除之前的多边形

        polygons.forEach(polygon => {
          const path = d3.path();
          polygon.forEach((point, i) => {
            const [x, y] = projectPoint(point); // 将地理坐标转换为像素坐标
            console.log(`${point} ${x}, ${y}`);
            if (i === 0) {
              path.moveTo(x, y); // 起点
            } else {
              path.lineTo(x, y); // 连线
            }
          });
          path.closePath(); // 闭合路径

          svg.append('path')
              .attr('d', path.toString())
              .attr('fill', 'lightblue')
              .attr('stroke', 'black')
              .attr('stroke-width', 1);
        });
      };

      // 初始绘制
      drawPolygons();

      // 添加拖动功能
      const drag = d3.drag()
          .on('drag', (event) => {
            console.log(event)
            const dx = event.dx;
            const dy = event.dy;
            const currentCenter = this.map.getCenter();
            const newCenter = this.map.containerPointToLatLng(
                this.map.latLngToContainerPoint(currentCenter).subtract([dx, dy])
            );
            this.map.setView(newCenter, this.map.getZoom(), { animate: false });
            drawPolygons(); // 重新绘制多边形
          });

      svg.call(drag); // 将拖动绑定到 SVG 容器

      // 监听地图缩放和拖动事件
      this.map.on('zoom', drawPolygons);
      this.map.on('move', drawPolygons);
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
<!--    <el-switch v-model="showPolygon" size="small"/>-->
    <div class="child-container" id="map-container-2"></div>
    <div class="child-container" id="svg-container-2"></div>
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

#map-container-2 {
  position: absolute; /* 或者 relative，具体取决于你的布局需求 */
  z-index: 1; /* 设置较小的 z-index 让它位于下面 */
}

#svg-container-2 {
  position: absolute; /* 或者 relative */
  z-index: 2; /* 设置较大的 z-index 让它位于上面 */
}
</style>