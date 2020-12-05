<template>
  <div id="app">
    <input type="button" style="background: none" value="+1 Gen" @click="updateGrid">
    <input type="button" style="background: none" value="Reset" @click="reset">
    <input type="button" style="background: none" value="Run" @click="run">
    <input type="button" style="background: none" value="Stop" @click="stop">
    <input type="button" style="background: none" value="Fast" @click="increaseSpeed">
    <input type="button" style="background: none" value="Slow" @click="decreaseSpeed">
    <input type="button" style="background: none" value="Zoom+" @click="increaseZoom">
    <input type="button" style="background: none" value="Zoom-" @click="decreaseZoom">
    <label for="height">Lines</label>
    <input type="number" id="height" style="background: none" :value="height" @click="changeHeight">
    <label for="width">Rows</label>
    <input type="number" id="width" style="background: none" :value="width" @click="changeWidth">
    <div>
      Speed : {{ speed }}
    </div>
    <div>
      Gen : {{ gen }}
    </div>
    <div v-if="displayed">
      <div>
        {{ displayed.x }}
      </div>
      <div>
        {{ displayed.y }}
      </div>
      <div>
        {{ displayed.xGrid }}
      </div>
      <div>
        {{ displayed.yGrid }}
      </div>
      <div>
        {{ displayed.color }}
      </div>
      <div>
        {{ displayed.nbAround }}
      </div>
    </div>
    <div v-if="grid.length" style="display: flex; margin-top: 30px" id="grid">
      <RecycleScroller
          style="display: flex; align-self: center; width: 100%"
          :items="grid"
          :item-size="30"
          key-field="id"
          v-slot="{ item }"
      >
        <div style="display: flex; margin-top: -30px">
          <div
              v-for="cell in item.line"
              :style="'width: ' + size + 'px; height: ' + size + 'px; border: 1px solid black; background: ' + cell.color"
              @click="cell.color = 'black'"
          ></div>
        </div>
      </RecycleScroller>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  },

  data() {
    return {
      grid: [],
      width: 50,
      height: 30,
      displayed: null,
      interval: null,
      speed: 1000,
      running: false,
      gen: 0,
      size: 20,
    }
  },

  created() {
    this.reset();
    history.pushState(null, null, location.href);
    window.onpopstate = function () {
      history.go(1);
    };
  },

  methods: {

    increaseZoom() {
      this.size++;
    },

    decreaseZoom() {
      this.size--;
    },

    changeHeight(event) {
      this.height = parseInt(event.target.value, 10);
      this.reset();
    },

    changeWidth(event) {
      this.width = parseInt(event.target.value, 10);
      this.reset();
    },

    increaseSpeed() {
      this.speed /= 2;
      this.clearAndRelaunch();
    },

    decreaseSpeed() {
      this.speed *= 2;
      this.clearAndRelaunch();
    },

    clearAndRelaunch() {
      if (this.running) {
        clearInterval(this.interval);
        this.$nextTick(() => {
          this.run();
        })
      }
    },

    stop() {
      this.running = false;
      clearInterval(this.interval);
    },

    run() {
      this.running = true;
      this.interval = setInterval(() => {
        this.updateGrid();
      }, this.speed);
    },

    reset() {
      this.gen = 0;
      this.stop();
      this.grid = [];
      let id = 0;
      for (let i = 0; i < this.height; i++) {
        let line = [];
        for (let j = 0; j < this.width; j++) {
          line.push({
            id: id,
            x: j,
            y: i,
            color: this.getRandomInt(20) % 11 === 0 ? 'black' : 'white',
          })
          id ++;
        }
        this.grid.push({
          id: i,
          line: line,
        });
      }
    },

    getRandomInt(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },

    updateGrid() {
      this.gen ++;
      let tmpGrid = [];
      this.grid.forEach((line, indexLine) => {
        let tmpLine = [];
        line.line.forEach((cell, indexCell) => {
          let tmpCell = {};
          let nbAliveAround = this.getNbAliveAround(indexLine, indexCell);

          if (cell.color === 'black') {
            if (nbAliveAround === 2 || nbAliveAround === 3) {
              tmpCell.color = 'black';
            } else {
              tmpCell.color = 'white';
            }
          } else {
            if (nbAliveAround === 3) {
              tmpCell.color = 'black';
            } else {
              tmpCell.color = 'white';
            }
          }

          tmpCell.x = cell.x;
          tmpCell.y = cell.y;
          tmpCell.id = cell.id;

          tmpLine.push(tmpCell);
        })
        tmpGrid.push({
          id: line.id,
          line: tmpLine,
        });
      })

      this.grid = tmpGrid;
    },

    getNbAliveAround(indexLine, indexCell) {
      let nbAliveAround = 0;

      if (this.grid[indexLine - 1]) {
        if (this.grid[indexLine - 1].line[indexCell - 1] && this.grid[indexLine - 1].line[indexCell - 1].color === 'black') {
          nbAliveAround++;
        }
        if (this.grid[indexLine - 1].line[indexCell] && this.grid[indexLine - 1].line[indexCell].color === 'black') {
          nbAliveAround++;
        }
        if (this.grid[indexLine - 1].line[indexCell + 1] && this.grid[indexLine - 1].line[indexCell + 1].color === 'black') {
          nbAliveAround++;
        }
      }

      if (this.grid[indexLine + 1]) {
        if (this.grid[indexLine + 1].line[indexCell - 1] && this.grid[indexLine + 1].line[indexCell - 1].color === 'black') {
          nbAliveAround++;
        }
        if (this.grid[indexLine + 1].line[indexCell] && this.grid[indexLine + 1].line[indexCell].color === 'black') {
          nbAliveAround++;
        }
        if (this.grid[indexLine + 1].line[indexCell + 1] && this.grid[indexLine + 1].line[indexCell + 1].color === 'black') {
          nbAliveAround++;
        }
      }

      if (this.grid[indexLine].line[indexCell - 1] && this.grid[indexLine].line[indexCell - 1].color === 'black') {
        nbAliveAround++;
      }
      if (this.grid[indexLine].line[indexCell + 1] && this.grid[indexLine].line[indexCell + 1].color === 'black') {
        nbAliveAround++;
      }

      return nbAliveAround;
    }
  },

  computed: {

  },

  watch: {

  }
}
</script>

<style>

</style>
