<template>
  <AudioPage />
</template>

<script>
import AudioPage from './views/AudioPage';

export default {
  name: 'App',
  components: {
    AudioPage,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
* {box-sizing: border-box; outline: 0;}
body {
  margin: 0;
  padding: 0;
  background-color: #1d1d1d;
}
</style>
