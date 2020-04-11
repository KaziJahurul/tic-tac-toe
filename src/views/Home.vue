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
