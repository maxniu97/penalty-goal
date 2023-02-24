<template>
  <div class="slider">
    <div class="power">
      <h1>kicking power</h1>
      <vue-Slider v-model="kickingPower"></vue-Slider>
    </div>
    <div class="angle">
      <h1>kicking angle</h1>
      <vue-Slider v-model="kickingAngle"></vue-Slider>
    </div>
  </div>
</template>

<script>
import VueSlider from "vue-slider-component";
//import 'vue-slider-component/theme/default.css'

export default {
  data() {
    return {
      kickingPower: 0,
      kickingAngle: 0,
    };
  },
  components: {
    VueSlider,
  },
};
</script>
]
<style scoped>

</style>
