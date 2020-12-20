<template>
  <div id="app">
    <div class="scores">POINTS: {{ points }} | HIGHEST: {{ maxPoints }}</div>
    <div class="board" @keyup.up="changeDirection('UP')">
      <div v-for="rowIndex in gridSize" :key="rowIndex" class="board--row">
        <Cell
          v-for="colIndex in gridSize" 
          :key="colIndex"
          :cell-size="cellSize"
          :grid-size="gridSize"
          :is-part-of-snake="isPartOfSnake(rowIndex, colIndex)"
          :is-apple="isApple(rowIndex, colIndex)"
        />
      </div>
    </div>

    <div class="game-actions">
      <button @click="startGame()">Start game</button>
      <button @click="stopGame()">Stop game</button>
    </div>

    <div class="description">
      To move the snake, use the arrow keys or the buttons below
    </div>

    <div class="game-controls">
      <div>
        <button @click="changeDirection('UP')">UP</button>
      </div>
      <div>
        <button class="lr-buttons" @click="changeDirection('LEFT')">LEFT</button>
        <button class="lr-buttons" @click="changeDirection('RIGHT')">RIGHT</button>
      </div>
      <div>
        <button @click="changeDirection('DOWN')">DOWN</button>
      </div>
    </div>
  </div>
</template>

<script>
import Cell from '@/components/Cell.vue';
import Vue from 'vue';

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
      maxPoints: 0
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
    getRandomInt() {
      let min = 0;
      let max = this.gridSize - 1;
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min) + min);
    },

    isAppleOnSnake(newAppleLocation) {
      for (let i = 0; i < this.snakeLocation.length; i++) {
        const partOfSnake = this.snakeLocation[i];
        if (newAppleLocation[0] === partOfSnake[0]
          && newAppleLocation[1] === partOfSnake[1])
          return true;
      }
      return false;
    },

    getRandomAppleLocation() {
      let newLocation = [this.getRandomInt(), this.getRandomInt()];
      while(this.isAppleOnSnake(newLocation)) {
        newLocation = [this.getRandomInt(), this.getRandomInt()];
      }
      this.appleLocation = newLocation;
    },

    isPartOfSnake(rowIndex, colIndex) {
      for (let i = 0; i < this.snakeLocation.length; i++) {
        const partOfSnake = this.snakeLocation[i];
        if (rowIndex - 1 === partOfSnake[0] && colIndex - 1 === partOfSnake[1])
          return true;
      }
      return false;
    },

    isApple(rowIndex, colIndex) {
      return rowIndex - 1 === this.appleLocation[0] && colIndex - 1 === this.appleLocation[1];
    },

    getNewSnakePartLocation(snakePart) {
      let partOfSnakeRow = snakePart[0];
      let partOfSnakeCol = snakePart[1];

      if (this.snakeDirection === 'RIGHT')
        partOfSnakeCol += 1;
      else if (this.snakeDirection === 'LEFT')
        partOfSnakeCol -= 1;
      else if (this.snakeDirection === 'DOWN')
        partOfSnakeRow += 1;
      else if (this.snakeDirection === 'UP')
        partOfSnakeRow -= 1;

      return [partOfSnakeRow, partOfSnakeCol];
    },

    isSnakeHeadOutOfGrid(snakeHead) {
      let row = snakeHead[0];
      let col = snakeHead[1];
      if (row >= this.gridSize || row < 0) return true;
      if (col >= this.gridSize || col < 0) return true;
      return false;
    },

    moveSnake() {
      let headIndex = this.snakeLocation.length - 1
      let snakeHead = this.snakeLocation[headIndex];
      
      let prevLocation = [snakeHead[0], snakeHead[1]];
      
      let newSnakeHead = this.getNewSnakePartLocation(snakeHead)

      if (this.isSnakeHeadOutOfGrid(snakeHead)) {
        this.stopGame();
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
    },

    stopGame() {
      clearInterval(this.gameInterval);
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

.board--row {
  display: flex;
  justify-content: center;
}

button {
  padding: 3px 8px;
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
  margin: 6px;
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
