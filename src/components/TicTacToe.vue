<template>
  <div class="tic-tac-toe">

    <div class="board">

      <div
              class="board_row"
              v-for="(row, row_index) in gameData.rows"
              :key="row_index"
      >
        <div
                class="board_col"
                v-for="(col, col_index) in row"
                :key="col_index"
                @click="tapClick"
                v-bind:data-row="row_index"
                v-bind:data-column="col_index"
        >

          {{ col }}

        </div> <!-- /.board_col -->

      </div> <!-- /.board_row -->

    </div> <!-- /.board -->


    <div class="status">
      <div v-if="gameData.finished">
        <p v-if="gameData.draw">It's a draw! Stalemate!</p>
        <p v-else>Finished! {{gameData.next}} wins!</p>
        <a v-on:click="restart">Restart</a>
      </div>
      <span v-else>Next go: {{gameData.next}}</span>
    </div> <!-- /.status -->

  </div>
</template>

<script>
  import io from "socket.io-client";
export default {
    name: 'TicTacToe',
    data() {
        return {
            gameData: {
                rows: [
                    ['', '', ''],
                    ['', '', ''],
                    ['', '', '']
                ],
                next: 'x',
                finished: false,
                draw: false,
            },
        }
    },
    created() {
      this.socket = io("http://localhost:3000");
      console.log(this.gameData);
    },
    methods: {
        tapClick(e) {
            if (!this.gameData.finished && e.target.innerText === '') {
                let rows = this.gameData.rows;
                rows[e.target.attributes['data-row'].value][e.target.attributes['data-column'].value] = this.gameData.next;
                this.gameData.rows = rows.slice(0);
                if (this.checkWinner()) {
                    this.gameData.finished = true;
                } else if (this.checkDraw()) {
                    this.gameData.draw = true;
                    this.gameData.finished = true;
                } else {
                    this.nextPlayer();
                }

                this.socket.emit("updateTapClick", this.gameData);
            }
        },
        checkWinner() {
            return (
                this.checkValues(this.gameData.rows[0]) ||
                this.checkValues(this.gameData.rows[1]) ||
                this.checkValues(this.gameData.rows[2]) ||
                this.checkValues([this.gameData.rows[0][0], this.gameData.rows[1][0], this.gameData.rows[2][0]]) ||
                this.checkValues([this.gameData.rows[0][1], this.gameData.rows[1][1], this.gameData.rows[2][1]]) ||
                this.checkValues([this.gameData.rows[0][2], this.gameData.rows[1][2], this.gameData.rows[2][2]]) ||
                this.checkValues([this.gameData.rows[0][0], this.gameData.rows[1][1], this.gameData.rows[2][2]]) ||
                this.checkValues([this.gameData.rows[0][2], this.gameData.rows[1][1], this.gameData.rows[2][0]]));
        },
        checkDraw() {
            return !this.gameData.finished &&
                (this.checkValuesPresent(this.gameData.rows[0]) &&
                    this.checkValuesPresent(this.gameData.rows[1]) &&
                    this.checkValuesPresent(this.gameData.rows[2]));
        },
        checkValues(values) {
            return this.checkValuesPresent(values) && this.checkValuesMatch(values);
        },
        checkValuesPresent(values) {
            return (values[0] !== '' && values[1] !== '' && values[2] !== '');
        },
        checkValuesMatch(values) {
            return (values[0] === values[1]) && (values[1] === values[2]);
        },
        nextPlayer() {
            this.gameData.next = (this.gameData.next === 'x' ? 'o' : 'x');
        },
        restart() {
            this.gameData.rows = [
                ['', '', ''],
                ['', '', ''],
                ['', '', '']
            ];
            this.gameData.finished = false;
            this.gameData.draw = false;
            this.nextPlayer();

            this.socket.emit("updateTapClick", this.gameData);
        },
    },
    mounted() {
      this.socket.on("gameData", data => {
         this.gameData = data;
      });
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  .board_row {
    clear: both;
  }
  .board_col {
    border: 1px solid #000;
    width: 50px;
    height: 35px;
    float: left;
    text-align: center;
    padding-top: 15px;
  }
  .status {
    clear: both;
  }
</style>
