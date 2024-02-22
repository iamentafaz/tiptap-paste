<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App" /> -->
  <TiptapEditor />
</template>

<script>
  // import HelloWorld from "./components/HelloWorld.vue";
  import TiptapEditor from './components/TipTapEditor.vue';

  export default {
    name: 'App',
    components: {
      // HelloWorld,
      TiptapEditor,
    },
  };
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    width: 100%;
    display: flex;
    justify-content: center;
  }
</style>
