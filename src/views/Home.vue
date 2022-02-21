<template>
  <div class="home">
    <!-- <div class="drawingtools">
      <drawingtools/>
    </div> -->
    <!-- <div class="olmap"> -->
      <olmap/>
    <!-- </div> -->
  </div>
</template>

<script>
import olmap from '../components/olmap.vue'
// import drawingtools from '../components/drawingtools.vue'
export default {
  name: 'Home',
  components: {
    olmap,
    // drawingtools
  }
}
</script>
<style lang="scss" scoped>
.home{
  width: 100%;
  height: 100%;
  // .drawingtools{
    // padding: 10px 0;
  // }
  // .olmap{
    // height: 100%;
  // }
}
</style>
