<template>
  <div class="sudoku">
    <h1 class="sudoku__title">Sudoku</h1>
    <div class="sudoku__grid">
      <div class="sudoku__row"
        :class="{ 'sudoku__row_border' : rowIdx === 2 || rowIdx === 5 }"
        v-for="(row, rowIdx) in puzzle"
        :key="rowIdx"  
      >
        <div class="sudoku__row-item"
          :class="{ 'sudoku__row-item_border' : colIdx === 2 || colIdx === 5 }"
          v-for="(item, colIdx) in row"
          :key="colIdx"
        >
          <div class="sudoku__row-item-content"
            :class="{ 
              'original': item.original,
              'active': activeItemX === rowIdx && activeItemY === colIdx
            }"
            @click="setItemActive(rowIdx, colIdx, item.original)"
          >
            {{item.value}}
          </div>
        </div>
      </div>
    </div>

    <div class="sudoku__buttons">
      
    </div>
  </div>
</template>

<script>
  import { sudoku } from 'sudoku.js/sudoku.js'

  export default {
    name: "Sudoku",
    data: function () {
      return {
        puzzle: [],
        difficulty: 'easy',
        activeItemX: -1,
        activeItemY: -1
      }
    },
    methods: {
      generatePuzzle () {
        const puzzleString = sudoku.generate(this.difficulty)
        //fill puzzle array with array of arrays of numbers
        this.puzzle = sudoku.board_string_to_grid(puzzleString)
          //convert to array of arrays of objects
          .map(row => {
            return row.map(el => {
              return {
                //remove value if it's not a number
                value: el !== '.' ? parseInt(el) : null,
                original: el !== '.'
              }
            })
          })
      },
      setItemActive (x, y, original) {
        if (original) {
          return
        }

        if (this.activeItemX === x && this.activeItemY === y) {
          this.activeItemX = this.activeItemY = -1
          return
        }

        this.activeItemX = x
        this.activeItemY = y 
      }
     },
     mounted () {
       this.generatePuzzle()
     }
  }
</script>

<style scoped lang="scss">
  $grid-width: 500px;
  $grid-gap: 3px;
  $border-radius: 7px;

  .sudoku {
    margin-top: 20px;

    &__title {
      font-weight: 400;
      text-align: center;
    }

    &__grid {
      width: $grid-width;
      margin: 0 auto;
      background-color: rgba(255, 99, 71, .7);
      border-radius: $border-radius;
    }

    &__row {
      display: grid;
      grid-template-columns: repeat(9, 1fr);

      &_border {
        border-bottom: 3px solid tomato;
      }
    }

    &__row-item {
      position: relative;

      &_border {
        border-right: 3px solid tomato;
      }

      &::before{
        content: "";
        display: block;
        padding-top: 100%;
      }
    }

    &__row-item-content {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(255, 255, 255, 0.7);
      display: grid;
      place-items: center;
      border: 1px solid white;
      cursor: pointer;
      // font-weight: 200;

      &::after {
        content: '';
        position: absolute;
        top: 1px;
        right: 1px;
        width: 6px;
        height: 6px;
        background-color: tomato;
        border-radius: 50%;
        border: 1px solid #fff;
        z-index: 100;
      }
    }
  }

  .original {
    cursor: default;

    &:after {
      content: none;
    }
  }

  .active {
    background: rgba(255, 255, 255, 0);
  }
</style>