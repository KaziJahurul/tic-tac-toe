<template>
  <div class="tic-tac-toe">

    <div class="board" v-if="gameData.gameStart">

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

      <div v-if="!clientPlayer" class="select-player">
        <h2>Select Player:</h2>
        <button v-if="!gameData.playerX" @click="setPlayer('x')">Player: X</button>
        <button v-if="!gameData.playerY" @click="setPlayer('o')">Player: O</button>
      </div>
      <h2 v-else>You are playing as player: {{clientPlayer}}</h2>

      <div class="game-finished" v-if="gameData.finished">
        <p v-if="gameData.draw">It's a draw! Stalemate!</p>
        <p v-else>Finished! {{gameData.next}} wins!</p>
        <button v-on:click="restart">Restart</button>
      </div>
      <span class="next-player" v-else-if="gameData.next">Next go: {{gameData.next}}</span>

      <div v-if="gameData.probX && gameData.probY" class="probablity">
        Odds of wining X: {{ gameData.probX.toFixed(2)}} %<br />
        Odds of wining O: {{ gameData.probY.toFixed(2)}} %<br />
        Odds of Stalemate: {{ gameData.probD.toFixed(2)}} %
      </div>

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
                next: '',
                finished: false,
                draw: false,
                playerX: '',
                playerY: '',
                probX: '',
                probY: '',
                probD: '',
                gameStart: false,
            },
            clientPlayer: '',
            winCombo: '',
        }
    },
    created() {
      this.socket = io("http://localhost:3000");
      if(this.getCookie('tictactoeplayer')){
          this.restart();
      }
    },
    methods: {
        setPlayer(e) {
            if(!this.getCookie('tictactoeplayer')){

                this.setCookie('tictactoeplayer', e, 0.00347);

                this.clientPlayer = this.getCookie('tictactoeplayer');

                if(e === 'x'){
                    this.gameData.playerX = this.getCookie('tictactoeplayer');
                } else if(e === 'o') {
                    this.gameData.playerY = this.getCookie('tictactoeplayer');
                }
                
                if (this.gameData.playerX === 'x' && this.gameData.playerY === 'o') {
                    this.gameData.gameStart = true;
                    this.gameData.next = 'x';
                }

                this.socket.emit("updateTapClick", this.gameData);

            }

        },
        tapClick(e) {
            if (!this.gameData.finished && e.target.innerText === '' && this.clientPlayer === this.gameData.next) {
                let rows = this.gameData.rows;
                rows[e.target.attributes['data-row'].value][e.target.attributes['data-column'].value] = this.gameData.next;
                this.gameData.rows = rows.slice(0);
                if (this.checkWinner()) {
                    this.gameData.finished = true;
                } else if (this.checkDraw()) {
                    this.gameData.draw = true;
                    this.gameData.finished = true;
                } else {
                    this.winProb();
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
        winProb() {
            this.winCombo = [
                this.gameData.rows[0],
                this.gameData.rows[1],
                this.gameData.rows[2],
                [this.gameData.rows[0][0], this.gameData.rows[1][0], this.gameData.rows[2][0]],
                [this.gameData.rows[0][1], this.gameData.rows[1][1], this.gameData.rows[2][1]],
                [this.gameData.rows[0][2], this.gameData.rows[1][2], this.gameData.rows[2][2]],
                [this.gameData.rows[0][0], this.gameData.rows[1][1], this.gameData.rows[2][2]],
                [this.gameData.rows[0][2], this.gameData.rows[1][1], this.gameData.rows[2][0]]
            ];

            let winX = this.winCombo.filter(el => el.includes('x') === true);
            let winXLeaveY = winX.filter(el => el.includes('o') !== true);
            this.gameData.probX = ((winXLeaveY.length / winX.length) / 2) * 100;

            let winY = this.winCombo.filter(el => el.includes('o') === true);
            let winYLeaveX = winY.filter(el => el.includes('x') !== true);
            this.gameData.probY = ((winYLeaveX.length / winY.length) / 2) * 100;

            this.gameData.probD = 100 - (this.gameData.probY + this.gameData.probX);

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
            this.gameData.next = 'x';
            this.gameData.playerX = '';
            this.gameData.playerY = '';
            this.gameData.probX = '';
            this.gameData.probY = '';
            this.gameData.probD = '';
            this.gameData.gameStart = false;

            this.socket.emit("refreshBoard", this.gameData);
        },
        clearClientCookie() {
            this.eraseCookie('tictactoeplayer');
            this.clientPlayer = '';
        },
        setCookie(cname, cvalue, exdays) {
            let d = new Date();
            d.setTime(d.getTime() + (exdays*24*60*60*1000));
            let expires = "expires="+ d.toUTCString();
            document.cookie = cname + "=" + cvalue + ";" + expires + ";path=/";
        },
        getCookie(cname) {
            let name = cname + "=";
            let decodedCookie = decodeURIComponent(document.cookie);
            let ca = decodedCookie.split(';');
            for(let i = 0; i <ca.length; i++) {
                let c = ca[i];
                while (c.charAt(0) === ' ') {
                    c = c.substring(1);
                }
                if (c.indexOf(name) === 0) {
                    return c.substring(name.length, c.length);
                }
            }
            return "";
        },
        eraseCookie(name) {
            document.cookie = name+'=; Max-Age=-99999999;';
        },
    },
    mounted() {
      this.socket.on("gameData", data => {
         this.gameData = data;
      });
      this.socket.on("refreshBoardAll", data => {
          this.clearClientCookie();
          this.gameData = data;
      });
    },
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  button{
    background: #fff;
    color: #14bdac;
    border: none;
    display: inline-block;
    cursor: pointer;
    padding: 15px 30px;
    margin: 20px;
    font-size: 18px;
    text-transform: uppercase;
  }
  .tic-tac-toe {
    background-color: #14bdac;
    width: 450px;
    margin: 0 auto;
    color: #fff;
    @media (max-width: 600px){
      width: 100%;
    }
  }
  .board{
    padding: 20px;

    .board_row {
      display: flex;
      border-bottom: 2px solid #fff;

      &:last-child{
        border-bottom: none;
      }

      .board_col {
        width: 33.33%;
        height: 135px;
        text-align: center;
        border-right: 2px solid #fff;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 100px;
        color: #fff;

        &:last-child{
          border-right: none;
        }

      }

    }

  }
  .status {
    padding: 30px 15px;

    .select-player{

      h2{
        margin: 0;
      }

    }
    .game-finished{
      p{
        font-weight: 700;
        font-size: 20px;
        margin: 10px 0 0;
      }
    }
    .next-player{
      font-size: 20px;
      font-weight: 700;
    }

  }
  .probablity{
    margin-top: 20px;
  }
</style>
