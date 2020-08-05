<template>
  <div id="app">
    <div class="simon-game-wrapper">
      <simon-game></simon-game>
    </div>
  </div>
</template>

<script>
import SimonGame from './components/simon_game.vue'

export default {
  name: 'App',
  components: {
    'simon-game': SimonGame
  }
}
</script>

<style lang="scss">

  .simon-game-wrapper {
    padding: 50px;
  }

  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
</style>
