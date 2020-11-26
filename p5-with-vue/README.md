# P5.js with Vue.js

[P5.js](https://p5js.org/) is a canvas tool for web development. The same libary used to make [Boid Boogie](https://boidboogie.com/).

I usually use [vue-p5](https://www.npmjs.com/package/vue-p5) to combine Vue.js with p5.js. It provides a simple component wrapper for a p5.js instance.

# Setting Up

Create the project:

```
vue create .
```

Install dependencies:

```
npm i vue-p5
```

# Creating a Graphic

Graphic.vue

```
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
```