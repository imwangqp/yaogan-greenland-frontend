<script>
import axios from "axios";
import _ from "lodash"

export default {
  name: "TestComponent",
  mounted () {
    this.initMap()
  },
  methods: {
    initMap() {

    }
  }
}
</script>

<template>
  <div id="map">

  </div>
</template>

<style scoped>
#map{
  width: 100%;
  height: 100%;
}
</style>