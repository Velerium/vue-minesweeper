<template>
  <div id="app">
    <div class="welcome">
      <div class="difficulty" v-if="this.difficulty == null">
        <button @click="setDiff(0)">Easy</button>
        <button @click="setDiff(1)">Hard</button>
      </div>
    </div>
    <div class="board" v-if="this.difficulty != null" :style="gridSize">
     <button class="new" @click="reset()">New Game</button>
     <div class="grid">
       <tileItem class="tileItem" @mineCheck="mineCheck1" v-for="(tile, index) in tiles" :key="index" :number="index" />
     </div>
    </div>
  </div>
</template>

<script>

import tileItem from '@/components/tileItem.vue';

export default {
  name: 'App',
  components: {
    tileItem,
  },

  data() {
    return {
      width: null,
      difficulty: null,
      sideLength: null,
      tileNum: null,
      tiles: [],
      mineNum: null,
      mines: [],
      flags: [],
      minesAround: 0,
      checkedTiles: [],
      toCheckTiles1: [],
      foreachFlag: false,
      clickCounter: 0,
    }
  },

  computed: {
    gridSize() {
      return {
        '--width': this.width + 'px',
      }
    },
  },

  methods: {

    genField() {
      for (var x = 0; x < this.tileNum; x++) {
        this.tiles.push(x);
      }
      while (this.mines.length < this.mineNum) {
        var z = Math.floor(Math.random() * this.tileNum);

        if (!this.mines.includes(z)) {
          this.mines.push(z);
        }
      }
    },

    setDiff(x) {
      this.difficulty = x;
      switch (x) {
        case 0:
          this.tileNum = 81;
          this.mineNum = 10;
          this.width = 225;
          this.sideLength = 9;
          break;
        case 1:
          this.tileNum = 256;
          this.mineNum = 40;
          this.width = 400;
          this.sideLength = 16;
          break;
      }
      this.genField();
    },

    reset() {
      this.setDiff(null);
      this.tileNum = null;
      this.tiles = [];
      this.mineNum = null;
      this.mines = [];
      this.flags = [];
      this.checkedTiles = [];
      this.toCheckTiles1 = [];
      this.foreachFlag = false;
    },

    mineCheck1(index) {
      this.clickCounter++;
      this.mineCheck2(index);
    },

    mineCheck2(index) {
      if (this.mines.includes(index)) {
        this.clickCounter--;
        this.gameOver();
        return;
      }

      if (this.mines.includes(index - 1)) {
        if (index % this.sideLength != 0) {
          this.minesAround++;
        }
      }

      if (this.mines.includes(index + 1)) {
        if (index % this.sideLength != this.sideLength - 1) {
          this.minesAround++;
        }
      }

      if (this.mines.includes(index - this.sideLength - 1)) {
        if (index % this.sideLength != 0 && index >= this.sideLength) {
          this.minesAround++;
        }
      }

      if (this.mines.includes(index - this.sideLength)) {
        if (index >= this.sideLength) {
          this.minesAround++;
        }
      }

      if (this.mines.includes(index - this.sideLength + 1)) {
        if (index % this.sideLength != this.sideLength - 1 && index >= this.sideLength) {
          this.minesAround++;
        }
      }

      if (this.mines.includes(index + this.sideLength - 1)) {
        if (index % this.sideLength != 0 && index < this.tiles.length - this.sideLength) {
          this.minesAround++;
        }
      }

      if (this.mines.includes(index + this.sideLength)) {
        if (index < this.tiles.length - this.sideLength) {
          this.minesAround++;
        }
      }

      if (this.mines.includes(index + this.sideLength + 1)) {
        if (index % this.sideLength != this.sideLength - 1 && index < this.tiles.length - this.sideLength) {
          this.minesAround++;
        }
      }

      if(this.minesAround != 0) {
        var tiles = document.getElementsByClassName("tileItem");
        tiles[index].innerText = `${this.minesAround}`;
      }

      if (this.minesAround == 0) {

        var square = document.getElementsByClassName("tileItem");
        square[index].style.backgroundColor = "#CCC"

        if (index % this.sideLength != 0) {
          if (!this.checkedTiles.includes(index - 1) && !this.toCheckTiles1.includes(index - 1)) {
            this.toCheckTiles1.push(index - 1)
          }
        }

        if (index % this.sideLength != this.sideLength - 1) {
          if (!this.checkedTiles.includes(index + 1) && !this.toCheckTiles1.includes(index + 1)) {
            this.toCheckTiles1.push(index + 1)
          }
        }

        if (index >= this.sideLength && index % this.sideLength != 0) {
          if (!this.checkedTiles.includes(index - this.sideLength - 1) && !this.toCheckTiles1.includes(index - this.sideLength - 1)) {
            this.toCheckTiles1.push(index - this.sideLength - 1)
          }
        }

        if (index >= this.sideLength) {
          if (!this.checkedTiles.includes(index - this.sideLength) && !this.toCheckTiles1.includes(index - this.sideLength)) {
            this.toCheckTiles1.push(index - this.sideLength)
          }
        }

        if (index % this.sideLength != this.sideLength - 1 && index >= this.sideLength) {
          if (!this.checkedTiles.includes(index - this.sideLength + 1) && !this.toCheckTiles1.includes(index - this.sideLength + 1)) {
            this.toCheckTiles1.push(index - this.sideLength + 1)
          }
        }

        if (index % this.sideLength != 0 && index < this.tiles.length - this.sideLength) {
          if (!this.checkedTiles.includes(index + this.sideLength - 1) && !this.toCheckTiles1.includes(index + this.sideLength - 1)) {
            this.toCheckTiles1.push(index + this.sideLength - 1)
          }
        }

        if (index < this.tiles.length - this.sideLength) {
          if (!this.checkedTiles.includes(index + this.sideLength) && !this.toCheckTiles1.includes(index + this.sideLength)) {
            this.toCheckTiles1.push(index + this.sideLength)
          }
        }

        if (index % this.sideLength != this.sideLength - 1 && index < this.tiles.length - this.sideLength) {
          if (!this.checkedTiles.includes(index + this.sideLength + 1) && !this.toCheckTiles1.includes(index + this.sideLength + 1)) {
            this.toCheckTiles1.push(index + this.sideLength + 1)
          }
        }
      }

      this.checkedTiles.push(index);
      this.minesAround = 0;

      while (!this.foreachFlag && this.toCheckTiles1.length != this.checkedTiles.length - this.clickCounter) {

        this.foreachFlag = true;
        this.toCheckTiles1.forEach(index => {
          if (!this.checkedTiles.includes(index)) {
            this.mineCheck2(index);
          }
        });
        this.foreachFlag = false;
      }
    },

    gameOver() {
      console.log('DED LMAO NOOB')
    },
  }
}

</script>

<style lang="scss">

@import './style/app.scss';

button {
  padding: 10px;
}

.welcome {
  display: flex;
  flex-direction: column;
  justify-content: center;

  .difficulty button {
    margin: 10px;
    font-size: 16px;
  }
}

.board {
  display: flex;
  justify-content: center;
  flex-direction: column;
  width: var(--width);
  height: 800px;

  .new {
    margin-bottom: 100px;
  }

  .grid {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;

    .tileItem {
      text-align: center;
      line-height: 25px;
      font-size: 17px;
      font-weight: bold;
      background-color: var(--bgColor);
    }
  }
}

</style>