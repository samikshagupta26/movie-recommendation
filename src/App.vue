<template>
  
  <div id="app">
    <!-- <div id="particles-background"></div> -->
    <Chatbot />
  </div>

</template>

<script>
import Chatbot from './components/ChatBot.vue';
// import { particlesJS } from 'particles.js';
export default {
  name: 'App',
  components: {
    Chatbot
  },
  // mounted() {
  //   // Initialize particles
  //   particlesJS.load('particles-background', '/particles.json', function() {
  //     console.log('Particles.js loaded');
  //   });
  // }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Handjet:wght@100..900&family=Protest+Guerrilla&display=swap');
html, body {
  margin: 0;
  height: 100%;
  overflow: hidden; /* Prevent scrolling */
}
#app {
  
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}

</style>