<template>
  <div id="app">
    <Sudoku />
  </div>
</template>

<script>

import Sudoku from '@/components/Sudoku'

export default {
  name: 'App',
  components: { Sudoku }
}
</script>

<style lang="scss">
  @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@200;300;400;600;700;800&display=swap');

  *, *::after, *::before {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  html, body {
    font-family: 'Nunito', sans-serif;
  }

  #app {
    font-family: 'Nunito', sans-serif;
    font-weight: 400;
    max-width: 900px;
    width: 100%;
    margin: 0 auto;
  }
</style>
