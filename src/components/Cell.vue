<template>
  <div
    class="board--row--cell"
    :style="{
      width: `${cellSize}px`,
      height: `${cellSize}px`,
      backgroundColor: getBackgroundColor
    }"
  ></div>
</template>

<script>
export default {
  props: {
    cellSize: {
      type: Number,
      default: 30
    },
    gridSize: {
      type: Number,
      default: 10
    },
    isPartOfSnake: {
      type: Boolean,
      default: false
    },
    isApple: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    getBackgroundColor: function() {
      if (this.isPartOfSnake) return '#A5FFEF';
      if (this.isApple) return '#CB1582';
      return 'transparent'
    }
  }
}
</script>

<style scoped>
.board--row--cell {
  border: 1px solid #1ED760;
  /* transition: all .05s ease-out; */
}
</style>