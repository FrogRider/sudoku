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
              'active': activeCellX === rowIdx && activeCellY === colIdx,
              'invalid': item.value && isInvalid(rowIdx, colIdx, item.value)
            }"
            @click="setItemActive(rowIdx, colIdx, item.original)"
          >
            {{item.value}}
          </div>
        </div>
      </div>
    </div>

    <div class="sudoku__funcs">
      <div class="sudoku__clear" @click="clear()">
        Clear
      </div>
      <p>{{this.steps}}</p>

      <div class="sudoku__reset" @click="reset()">
        Reset
      </div>

      <div class="sudoku__undo" @click="undo()">
        Undo
      </div>
    </div>

    <div class="sudoku__buttons">
      <div class="sudoku__button"
        v-for="number in Array(9).keys()"
        :key="number"
      >
        <div class="sudoku__button-inner"
          @click="addNumber(number + 1)"
        >
          {{ number + 1 }}
        </div>
      </div>
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
        difficulty: 'normal',
        //selected cell's coordinates
        activeCellX: -1,
        activeCellY: -1,
        history: [],
        steps: 0
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
          //unables to change default numbers
          return
        }

        if (this.activeCellX === x && this.activeCellY === y) {
          //if item was selected and then clicked one more time - deselect it
          this.activeCellX = this.activeCellY = -1
          return
        }

        //select scell
        this.activeCellX = x
        this.activeCellY = y 
      },
      addNumber (num) {
        //add a number to selected cell
        if (this.activeCellX !== -1 && this.activeCellY !== -1) {
          this.steps++
          this.puzzle[this.activeCellX][this.activeCellY].value = num
          this.history.push({
            x: this.activeCellX,
            y: this.activeCellY,
            value: num
          })
        }

        if(this.isGameComplete()) {
          console.log(`Success! \n ${this.getSteps} steps was required`)
          alert(`Success! \n ${this.getSteps} steps was required`)
          this.reset()
        }
      },
      undo () {
        if(this.history.length > 1) {
          //erase current value
          const valueToDelete = this.history.pop()
          this.puzzle[valueToDelete.x][valueToDelete.y].value = null
          //backup previous value from history
          const lastChange = this.history[this.history.length - 1]
          //set backuped value to the current
          this.activeCellX = lastChange.x
          this.activeCellY = lastChange.y
          this.puzzle[lastChange.x][lastChange.y].value = lastChange.value
          this.steps++
        } 
      },
      clear () {
        this.puzzle.map(row => {
          //loop through all the rows
          row.map(el => {
            //loop through all the elements in current row
            if(!el.original) {
              //reset all the numbers user has inputed
              el.value = null
             }
           })
         })
         //remove selection from the cell
         this.activeCellX = this.activeCellY = -1
         this.steps = 0
       },
       reset () {
         this.activeCellX = this.activeCellY = -1
         //generate new puzzle
         this.generatePuzzle()
         this.steps = 0
       },
       isInvalid(x, y, val) {
        if (!val) return true

        for (let c = 0; c < 9; c += 1) {
          //check for identical values in:
          if (this.puzzle[x][c].value === val && c !== y) return true
          //row
          if (this.puzzle[c][y].value === val && c !== x) return true
          //in column
        }

        const rowStart = Math.floor(x / 3) * 3
        const colStart = Math.floor(y / 3) * 3

        for (let r = rowStart; r < rowStart + 3; r += 1) {
          //in 3*3 square
          for (let c = colStart; c < colStart + 3; c += 1) {
            if (this.puzzle[r][c].value === val && !(r === x && c === y)) {
              return true
            }
          }
        }

        return false
       },
       isGameComplete () {
         let errors = []
         this.puzzle.map((row, rowId) => {
           row.map((el, colId) => {
            //  console.log(`${rowId} - ${colId} - ${el.value}`)
             errors.push(this.isInvalid(rowId, colId, el.value))
           })
         })

         if (errors.indexOf(true) === -1) {
           this.steps = 0
           return true
         } 
       }
     },
     mounted () {
       this.generatePuzzle()
     },
     computed: {
      getSteps: function () {
        return this.steps
      }
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
      max-width: $grid-width;
      width: 100%;
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

    &__buttons {
      max-width: $grid-width;
      width: 100%;
      margin: 10px auto 0;
      display: grid;
      grid-template-columns: repeat(9, 1fr);
    }

    &__button {
      position: relative;

      &::before {
        content: "";
        display: block;
        padding-top: 100%;
      }
    }

    &__button-inner {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      display: grid;
      place-items: center;
      background-color: red;
      border: 1px solid white;
      cursor: pointer;
    }

    &__funcs {
      display: flex;
      justify-content: space-between;
      max-width: $grid-width;
      width: 100%;
      margin: 0 auto;
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

  .invalid {
    background-color: red;
  }
</style>