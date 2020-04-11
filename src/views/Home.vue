<template>
  <div class="home">

    <div class="board">

      <div
              class="board_row"
              v-for="(row, row_index) in rows"
              :key="row_index"
      >
        <div
                class="board_col"
                v-for="(col, col_index) in row"
                :key="col_index"
                v-on:click="tapClick"
                v-bind:data-row="row_index"
                v-bind:data-column="col_index"
        >

          {{ col }}

        </div> <!-- /.board_col -->

      </div> <!-- /.board_row -->

    </div> <!-- /.board -->


    <div class="status">
      <span v-if="finished">Finished! {{next}} wins!</span>
      <span v-else>Next go: {{next}}</span>
    </div> <!-- /.status -->

  </div>
</template>

<script>
// @ is an alias to /src

export default {
    name: 'Home',
    components: {

    },
    data() {
      return {
          rows: [
              ['', '', ''],
              ['', '', ''],
              ['', '', '']
          ],
          next: 'x',
          finished: false
      }
    },
    methods: {
        tapClick(e) {
            if (!this.finished && e.target.innerText === '') {
                let rows = this.rows;
                rows[e.target.attributes['data-row'].value][e.target.attributes['data-column'].value] = this.next;
                this.rows = rows.slice(0);
                if (this.checkWinner()) {
                    this.finished = true;
                } else {
                    this.next = (this.next === 'x' ? 'o' : 'x');
                }
            }
        },
        checkWinner() {
            return (
                this.checkValues(this.rows[0]) ||
                this.checkValues(this.rows[1]) ||
                this.checkValues(this.rows[2]) ||
                this.checkValues([this.rows[0][0], this.rows[1][0], this.rows[2][0]]) ||
                this.checkValues([this.rows[0][1], this.rows[1][1], this.rows[2][1]]) ||
                this.checkValues([this.rows[0][2], this.rows[1][2], this.rows[2][2]]) ||
                this.checkValues([this.rows[0][0], this.rows[1][1], this.rows[2][2]]) ||
                this.checkValues([this.rows[0][2], this.rows[1][1], this.rows[2][0]]));
        },
        checkValues(values) {
            return (values[0] !== '' && values[1] !== '' && values[2] !== '' && (values[0] === values[1]) && (values[1] === values[2]));
        },

    },
}
</script>

<style lang="scss">
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
