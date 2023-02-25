<template>
  <div class="slider">
    <span class="text">Power</span>
    <el-slider v-model="kickingPower"></el-slider>
  </div>
  <div class="slider">
    <span class="text">Angle</span>
    <el-slider v-model="kickingAngle"></el-slider>
  </div>
</template>

<script>


export default {
  data() {
    return {
      kickingPower: 0,
      kickingAngle: 0,
    };
  },
  
};
</script>

<style scoped>
.slider {
  display: flex;
  align-items: center;
 
}
.slider .el-slider {
  margin-top: 0;
  margin-left: 0px;

}
.slider .text {
  font-size: 24px;
  color: black;
  line-height: 44px;
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin-bottom: 0;
 
}
.slider .text + .el-slider {
  flex: 0 0 70%;

}
</style>