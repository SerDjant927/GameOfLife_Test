<template>
  <div>
    <main class="game">
      <canvas ref="gameCanvas" width="1280" height="1024" class="game-area"></canvas>
    </main>
    <footer class="bar">
      <button class="button button_random" @click="randomize">Random</button>
      <button class="button button_start" @click="startStop">Start/Stop</button>
      <button class="button button_clear" @click="clear">Clear</button>
    </footer>
  </div>
</template>

<script>
export default {
  mounted() {
    const NEIGHBORING_CELLS = [
      [-1, -1], [-1, 0], [-1, 1], [1, -1],
      [1, 0], [1, 1], [0, 1], [0, -1]
    ];

    class Life {
      constructor(canvas, opts) {
        opts = opts || {};

        this.cellSize = opts.cellSize || 10;
        this.x = opts.x || 70;
        this.y = opts.y || 70;

        this.canvas = canvas;
        this.canvas.addEventListener('click', this._changePoint.bind(this));

        this.ctx = canvas.getContext('2d');
        this.ctx.fillStyle = opts.color || '#444';

        this.clear();
      }

      _changePoint(event) {
        const x = Math.floor((event.pageX - event.currentTarget.offsetLeft) / this.cellSize);
        const y = Math.floor((event.pageY - event.currentTarget.offsetTop) / this.cellSize);

        this.stop();
        this.area[y][x] = !this.area[y][x];
        this._draw();
      }

      _fillRect(x, y) {
        this.ctx.fillRect(x * this.cellSize, y * this.cellSize, this.cellSize, this.cellSize);
      }

      _draw() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.area.forEach((line, y) => line.forEach((cell, x) => cell && this._fillRect(x, y), this), this);
      }

      _isAlive(pointX, pointY) {
        const count = NEIGHBORING_CELLS.reduce((sum, diff) => {
          const x = pointX - diff[0];
          const y = pointY - diff[1];

          return (x < 0 || x >= this.area[0].length || y < 0 || y >= this.area.length) ?
            sum : sum + this.area[y][x];
        }, 0);

        return this.area[pointY][pointX] ? (count === 2 || count === 3) : count === 3;
      }

      addFigure(x, y, matrix) {
        if (x + matrix[0].length > this.x || y + matrix.length > this.y) return;

        for (let index = 0; index < matrix.length; index++) {
          this.area[index + y] = this.area[index + y].slice(0, x)
            .concat(matrix[index], this.area[index + y].slice(matrix[index].length, this.x));
        }

        this._draw();
      }

      start(tick) {
        if (this.game) return;

        this.game = setInterval(self => {
          self.area = self.area.map((row, y) => row.map((cell, x) => self._isAlive(x, y)));
          self._draw();
        }, tick || 100, this);
      }

      stop() {
        if (this.game) {
          clearInterval(this.game);
          this.game = null;
        }
      }

      clear() {
        this.area = Array(this.y).fill(0).map(() => Array(this.x).fill(0));
        this._draw();
      }
    }

    const gameCanvas = this.$refs.gameCanvas;
    const cellSize = /Android|iPhone|Mobile/gi.test(navigator.userAgent) ? 20 : 10;
    const area = document.querySelector('.game');
    const x = Math.round(area.offsetWidth / cellSize);
    const y = Math.round(area.offsetHeight / cellSize);

    gameCanvas.width = area.offsetWidth;
    gameCanvas.height = area.offsetHeight;

    const game = new Life(gameCanvas, { cellSize, x, y });

    const startStop = () => {
      if (game.game) {
        game.stop();
      } else {
        game.start();
      }
    };

    const randomize = () => {
      game.addFigure(
        0, 0, Array(y).fill(Array(x).fill(0))
          .map(line => line.map(() => (Math.random() / Math.random()) > 0.3 ? 0 : 1))
      );
    };

    const clear = () => {
      game.stop();
      game.clear();
    };

    this.startStop = startStop;
    this.randomize = randomize;
    this.clear = clear;
  }
};
</script>

<style scoped>
canvas{
  height: 90vh;
  width: 90vw;
}
html,
body {

  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}

body {
  user-select: none;
  display: flex;
  flex-direction: column;
}

.game {
  flex-grow: 1;
  overflow: hidden;
}

.bar {
  width: 100%;
  height: 40px;
  bottom: 0;
  display: flex;
  align-items: center;
  padding: 0 16px;
}

.mobile .bar {
  justify-content: center;
}

 .button {
        border: 0;
        padding: 10px 14px;
        border-radius: 8px;
        margin-right: 16px;
        cursor: pointer;
        background-color: #bbff00;
        color: #3d3d3d;
        font-size: 16px;
        font-weight: 700;
        display: flex;
        align-items: center;
        justify-content: center;
    }

</style>
