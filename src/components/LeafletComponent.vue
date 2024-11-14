<script>
import axios from "axios";
import _ from "lodash"
// import '../util/baidu.js'
import 'proj4leaflet'
import * as d3 from 'd3'
import proj4 from "proj4"
import {wgs84togcj02, gcj02tobd09} from 'coordtransform'
import * as L from "leaflet"
import 'leaflet.chinatmsproviders'

export default {
  name: "LeafletComponent",
  mounted() {
   this.initMap()
    this.initSvg()
  },
  data(){
    return{
      map: null
    }
  },
  methods: {
    initMap(){
      const map = L.map('map-container', {
        minZoom: 3,
        maxZoom: 18,
        center: [23.153021511848955, 114.49112893364193],
        zoom: 13,
        attributionControl: false
      })
      this.map = map
      L.tileLayer.chinaProvider('TianDiTu.Normal.Map', {maxZoom:18,minZoom:5}).addTo(map);

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
    initSvg(){
      const width = document.getElementById('svg-container').offsetWidth,
          height = document.getElementById('svg-container').offsetHeight
      const drag = d3.drag()
          .on('drag', e => {
            const dx = e.dx;
            const dy = e.dy;

            // Calculate new center based on drag movement
            const currentCenter = this.map.getCenter();
            const newCenter = this.map.containerPointToLatLng(this.map.latLngToContainerPoint(currentCenter).subtract([dx, dy]));

            this.map.setView(newCenter, this.map.getZoom(), {animate: false});
          })
      const svg = d3.select('#svg-container')
          .append('svg')
          .attr('width', width)
          .attr('height', height)
          .on('mousewheel', e => {
            if (e.wheelDelta > 0){
              this.map.zoomOut()
            } else if (e.wheelDelta < 0) {
              this.map.zoomIn()
            }
          })
          .call(drag)
    }
  }
}
</script>

<template>
  <div class="full-container">
    <div class="child-container" id="map-container"></div>
    <div class="child-container" id="svg-container"></div>
  </div>
</template>

<style scoped>
.child-container{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.full-container{
  position: relative;
  width: 100%;
  height: 100%;
}
</style>