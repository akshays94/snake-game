<template>
    <div
        class="board--row--cell"
        :style="{
            width: `${cellSize}px`,
            height: `${cellSize}px`,
            border: `1px solid ${isGameOver ? 'red': '#1ED760'}`,
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
        },
        isGameOver: {
            type: Boolean,
            default: false,
        }
    },
    computed: {
        getBackgroundColor: function() {
            if (this.isPartOfSnake) {
                if (this.isGameOver) {
                    return 'pink';
                }
                return '#A5FFEF';
            } else if (this.isApple) {
                if (this.isGameOver) {
                    return 'transparent';
                }
                return '#CB1582';
            }
            return 'transparent'
        }
    }
}
</script>

<style scoped>
.board--row--cell {
    border: 1px solid #1ED760;
}
</style>