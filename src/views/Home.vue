<template>
  <div class="home">

  <TicTacToe></TicTacToe>

  </div>
</template>

<script>
// @ is an alias to /src
import TicTacToe from '../components/TicTacToe';
export default {
    name: 'Home',
    components: {
        TicTacToe
    },
}
</script>

<style lang="scss">
  .home{
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
  }
</style>
