<template>
  <vue-p5 v-on="{ setup, draw }" />
</template>

<script>
import VueP5 from 'vue-p5';

export default {
  name: 'Graphic',
  components: {
    VueP5,
  },
  methods: {
    setup(sketch) {
      const width = 200;
      const height = 300;

      sketch.resizeCanvas(width, height);

      // ...More Setup
    },
    draw(sketch) {
      sketch.clear();
      sketch.background('rgba(10,10,10,1)');

      // ...More Drawing
    },
  },
};
</script>

<style scoped>
</style>