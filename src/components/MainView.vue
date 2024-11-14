<script>

import {defineComponent} from "vue";
import LeafletComponent from "./LeafletComponent.vue";
import TestComponent from "./TestComponent.vue";
import TimeController from "./TimeController.vue";

export default defineComponent({
  components: {TimeController, TestComponent, LeafletComponent}
})
</script>

<template>
  <div class="container">
    <div class="box"><LeafletComponent /></div>
    <div class="box"><TimeController /></div>
    <div class="box">Bottom Left</div>
    <div class="box">Bottom Right</div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-wrap: wrap; /* 允许子元素换行 */
  width: 100%; /* 或指定一个固定宽度 */
  height: 100vh; /* 或指定一个固定高度 */
}

.box {
  flex: 0 0 50%; /* 每个子元素占据 50% 的宽度 */
  height: 50%; /* 每个子元素占据 50% 的高度 */
  box-sizing: border-box;
  border: 1px solid #000;
  text-align: center;
}

.top-left {
  background-color: lightblue;
}

.top-right {
  background-color: lightgreen;
}

.bottom-left {
  background-color: lightcoral;
}

.bottom-right {
  background-color: lightyellow;
}

</style>