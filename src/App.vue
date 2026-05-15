<template>
  <div id="app">
    <!-- Displays the standalone jump game component -->
    <JumpGame />
  </div>
</template>

<script>
// Imports the clean standalone component
import JumpGame from "./components/jumpGame.vue";

export default {
  name: "App",
  components: {
    JumpGame
  }
};
</script>

<style>
/* Resets page layout and centers the game screen perfectly */
body {
  margin: 0;
  padding: 0;
  background-color: #fafafa;
}

#app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
</style>
