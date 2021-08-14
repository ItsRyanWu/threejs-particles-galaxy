<template>
  <Particles/>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Particles from './components/Particles.vue'

export default defineComponent({
  name: 'App',
  components: {
    Particles
  }
})
</script>

<style>

body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
