<template>
    <div id="app">
        <section class="scores">
            POINTS: {{ points }} | HIGHEST: {{ maxPoints }}
        </section>

        <section class="board">
            <div class="game-over" v-if="isGameOver">
                GAME OVER
            </div>
            
            <div v-for="rowIndex in gridSize" :key="rowIndex" class="board--row">
                <Cell
                    v-for="colIndex in gridSize" 
                    :key="colIndex"
                    :cell-size="cellSize"
                    :grid-size="gridSize"
                    :is-part-of-snake="isCellAPartOfSnake(rowIndex, colIndex)"
                    :is-apple="isCellAnApple(rowIndex, colIndex)"
                    :is-game-over="isGameOver"
                />
            </div>
        </section>

        <section class="game-actions">
            <button v-if="!isGameStarted && !isGameOver" @click="startGame()">
                Start game
            </button>
            <button v-if="isGameOver" @click="stopGame()">
                Play Again
            </button>
        </section>

        <section class="description">
            To move the snake, use the arrow keys or the buttons below
        </section>

        <section class="game-controls">
            <div>
                <button @click="changeDirection('UP')">
                    UP
                </button>
            </div>
            <div>
                <button class="lr-buttons" @click="changeDirection('LEFT')">
                    LEFT
                </button>
                <button class="lr-buttons" @click="changeDirection('RIGHT')">
                    RIGHT
                </button>
            </div>
            <div>
                <button @click="changeDirection('DOWN')">
                    DOWN
                </button>
            </div>
        </section>
    </div>
</template>

<script>
import Vue from 'vue';
import Cell from '@/components/Cell.vue';

export default {
    name: 'App',
    components: {
        Cell
    },
    data() {
        return {
            gridSize: 10,
            cellSize: 30,
            snakeLocation: [
                [0, 0],
                [0, 1],
                [0, 2]
            ],
            appleLocation: [-1, -1],
            snakeDirection: 'RIGHT',
            gameInterval: null,
            points: 0,
            maxPoints: 0,

            isGameStarted: false,
            isGameOver: false
        }
    },
    created() {
        window.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowUp') {
                this.changeDirection('UP');
                if(!this.gameInterval) {
                    this.startGame();
                }
            } else if (e.key === 'ArrowDown') {
                this.changeDirection('DOWN');
                if(!this.gameInterval) {
                    this.startGame();
                }
            } else if (e.key === 'ArrowLeft') {
                this.changeDirection('LEFT');
                if(!this.gameInterval) {
                    this.startGame();
                }
            } else if (e.key === 'ArrowRight') {
                this.changeDirection('RIGHT');
                if(!this.gameInterval) {
                    this.startGame();
                }
            } else if (e.key === 'Escape') {
                this.stopGame();
            }
        });

        this.getRandomAppleLocation();
    },

    methods: {
        isCellAPartOfSnake(rowIndex, colIndex) {
            for (const partOfSnake of this.snakeLocation) {
                if (rowIndex - 1 === partOfSnake[0]
                    && colIndex - 1 === partOfSnake[1]) {
                    return true;
                }
            }
            return false;
        },

        isCellAnApple(rowIndex, colIndex) {
            return rowIndex - 1 === this.appleLocation[0]
                && colIndex - 1 === this.appleLocation[1];
        },

        getRandomInt() {
            let min = 0;
            let max = this.gridSize - 1;
            min = Math.ceil(min);
            max = Math.floor(max);
            return Math.floor(Math.random() * (max - min) + min);
        },

        isNewAppleLocationOnSnake(newAppleLocation) {
            for (const partOfSnake of this.snakeLocation) {
                if (newAppleLocation[0] === partOfSnake[0]
                    && newAppleLocation[1] === partOfSnake[1]) {
                    return true;
                }
            }
            return false;
        },

        getRandomAppleLocation() {
            let newLocation = [
                this.getRandomInt(),
                this.getRandomInt()
            ];

            while(this.isNewAppleLocationOnSnake(newLocation)) {
                newLocation = [
                    this.getRandomInt(),
                    this.getRandomInt()
                ];
            }

            this.appleLocation = newLocation;
        },

        getNewSnakeHead(oldSnakeHead) {
            let oldSnakeHeadRow = oldSnakeHead[0];
            let newSnakeHeadCol = oldSnakeHead[1];

            if (this.snakeDirection === 'RIGHT')
                newSnakeHeadCol += 1;
            else if (this.snakeDirection === 'LEFT')
                newSnakeHeadCol -= 1;
            else if (this.snakeDirection === 'DOWN')
                oldSnakeHeadRow += 1;
            else if (this.snakeDirection === 'UP')
                oldSnakeHeadRow -= 1;

            return [oldSnakeHeadRow, newSnakeHeadCol];
        },

        isSnakeHeadOutOfGrid(snakeHead) {
            let row = snakeHead[0];
            let col = snakeHead[1];
            if (row >= this.gridSize || row < 0) return true;
            if (col >= this.gridSize || col < 0) return true;
            return false;
        },

        isSnakeAteItself(newSnakeHead) {
            for (const snakePart of this.snakeLocation) {
                if (newSnakeHead[0] === snakePart[0]
                    && newSnakeHead[1] === snakePart[1]) {
                        return true;
                }
            }
            return false;
        },

        moveSnake() {
            let headIndex = this.snakeLocation.length - 1
            let snakeHead = this.snakeLocation[headIndex];
            
            let prevLocation = [snakeHead[0], snakeHead[1]];
            
            let newSnakeHead = this.getNewSnakeHead(snakeHead)

            if (this.isSnakeHeadOutOfGrid(newSnakeHead)
                || this.isSnakeAteItself(newSnakeHead)) {

                this.markGameOver();

            } else {
                
                Vue.set(this.snakeLocation, headIndex, newSnakeHead);

                const isAppleEaten =
                    newSnakeHead[0] === this.appleLocation[0]
                    && newSnakeHead[1] === this.appleLocation[1]

                if (isAppleEaten) {
                    this.getRandomAppleLocation();
                    this.points += 1;
                }

                for (let i = this.snakeLocation.length - 2; i >= 0; i--) {
                    let currentSnakeLocation = this.snakeLocation[i];
                    let oldCurrentSnakeLocation 
                        = [currentSnakeLocation[0], currentSnakeLocation[1]];

                    Vue.set(this.snakeLocation, i, prevLocation);

                    prevLocation = oldCurrentSnakeLocation;
                }

                if (isAppleEaten) {
                    this.snakeLocation.unshift(prevLocation);
                }
            }
        },

        startGame() {
            this.gameInterval = setInterval(this.moveSnake, 200);
            this.isGameStarted = true;
            this.isGameOver = false;
        },

        markGameOver() {
            clearInterval(this.gameInterval);
            this.isGameStarted = false;
            this.isGameOver = true;
        },

        stopGame() {
            clearInterval(this.gameInterval);
            this.isGameStarted = false;
            this.isGameOver = false;

            this.gameInterval = null;

            this.snakeLocation = [
                [0, 0],
                [0, 1],
                [0, 2]
            ];

            this.getRandomAppleLocation();
            this.snakeDirection = 'RIGHT';

            if (this.maxPoints < this.points) {
                this.maxPoints = this.points
            }

            this.points = 0;
        },

        changeDirection(newDirection) {
            this.snakeDirection = newDirection;
        }
    }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Source+Code+Pro:wght@200;500&display=swap');

body {
    background: #2C2C32;
}

#app {
    font-family: 'Source Code Pro', Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    margin: 0 auto;
}

.board {
    position: relative;
}

.game-over {
    color: white;
    font-size: 2em;
    font-weight: bolder;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-50%);
}

.board--row {
    display: flex;
    justify-content: center;
}

button {
    padding: 8px 12px;
    font-weight: normal;
    font-size: 0.8em;
    font-family: 'Source Code Pro';
    background: #1ED760;
    cursor: pointer;
    border-radius: 10px;
}

.game-actions {
    margin: 12px;
}

.game-actions > button {
    margin: 0 8px;
}

.game-controls {
    margin-top: 24px;
}

.game-controls button {
    width: 80px;
}

.game-controls button.lr-buttons {
    margin: 8px;
}

.scores {
    color: #A5FFEF;
    font-weight: bold;
    font-size: 1.09em;
    margin: 12px 0;
}

.description {
    color: lightgray;
}
</style>
