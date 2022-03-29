<template>
  <div id="app">
    <div class="welcome" v-if="this.difficulty == null">
      <span>Select the difficulty!</span>
      <div class="difficulty">
        <button @click="setDiff(0)">Easy</button>
        <button @click="setDiff(1)">Hard</button>
      </div>
    </div>
    <div class="board" v-if="this.difficulty != null" :style="gridSize">
      <button class="new" @click="reset()">New Game</button>
      <div class="grid" id="grid">
        <tileItem class="tileItem" @mineCheck="mineCheck1" v-for="(tile, index) in tiles" :key="index" :number="index" />
      </div>
      <div class="modes" id="modes">
        <button class="modeButton" @click="changeModeSweep()">⛏️</button>
        <button class="hintButton" @click="hint()">💡</button>
        <button class="modeButton" @click="changeModeFlag()">🚩</button>
      </div>
      <div class="hintsLeft">
        <span>Hints used: {{this.hints.length}}/{{this.hintLimit}}</span>
      </div>
      <div class="flagsLeft">
        <span :style="flagSpan">Flags used: {{this.flags.length}}/{{this.mineNum}}</span>
      </div>
    </div>

    <modal-bomb :show="showModalBomb" @close="showModalBomb = false" @disable="disable">
      <template #header>
        <h3>Ouch!</h3>
      </template>
    </modal-bomb>

    <modal-win :show="showModalWin" :hints="hints"
    :hintLimit="hintLimit" @close="showModalWin = false" @disable="disable">
      <template #header>
        <h3>Wow!</h3>
      </template>
    </modal-win>
  </div>
</template>

<script>

import tileItem from '@/components/tileItem.vue';
import ModalBomb from '@/components/ModalBomb.vue'
import ModalWin from '@/components/ModalWin.vue'

export default {
  name: 'App',
  components: {
    tileItem,
    ModalBomb,
    ModalWin,
  },

  data() {
    return {
      mode: 'sweep',
      width: null,
      flagColor: 'black',
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
      loopCounter: 0,
      hints: [],
      hintLimit: 0,
      showModalBomb: false,
      showModalWin: false,
    }
  },

  computed: {
    gridSize() {
      return {
        '--width': this.width + 'px',
      }
    },
    flagSpan() {
      return {
        '--color': this.flagColor,
      }
    }
  },

  methods: {

    genField() {
      for (var x = 0; x < this.tileNum; x++) {
        this.tiles.push(x);
      }
      this.genMines();
    },

    genMines() {
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
          this.hintLimit = 3
          break;
        case 1:
          this.tileNum = 256;
          this.mineNum = 40;
          this.width = 400;
          this.sideLength = 16;
          this.hintLimit = 1
          break;
      }
      this.genField();
    },

    changeModeSweep() {
      if (this.mode == 'flag') {
        this.mode = 'sweep';
      }
    },

    changeModeFlag() {
      if (this.mode == 'sweep') {
        this.mode = 'flag';
      }
    },

    hint() {
      if (this.hints.length == this.hintLimit) {
        return;
      }
      
      var randomTile;
      do {
        randomTile = Math.floor(Math.random() * this.tileNum);
      } while (this.checkedTiles.includes(randomTile) || this.mines.includes(randomTile));

      var allTiles = document.getElementsByClassName("tileItem");
      allTiles[randomTile].style.backgroundColor = 'yellow';
      this.hints.push(randomTile);
    },

    reset() {
      this.mode = 'sweep';
      this.width = null;
      this.difficulty = null;
      this.sideLength = null;
      this.tileNum = null;
      this.tiles = [];
      this.mineNum = null;
      this.mines = [];
      this.flags = [];
      this.minesAround = 0;
      this.checkedTiles = [];
      this.toCheckTiles1 = [];
      this.foreachFlag = false;
      this.clickCounter = 0;
      this.loopCounter = 0;
      this.hints = [];
      this.hintLimit = 0;
      this.showModalBomb = false;
      this.showModalWin = false;
    },

    disable() {
      var grid = document.getElementById("grid");
      var modes = document.getElementById("modes");

      grid.style.pointerEvents = "none";
      modes.style.pointerEvents = "none";
    },

    mineCheck1(index) {

      var tiles = document.getElementsByClassName("tileItem");

      if (this.mode == 'flag') {
        if (!this.flags.includes(index)) {
          if (tiles[index].innerText == '' && !this.checkedTiles.includes(index)) {
            this.flags.push(index)
            tiles[index].innerText = '🚩'
          }
        } else {
          var remove = this.flags.indexOf(index)
          this.flags.splice(remove, 1);
          tiles[index].innerText = ''
        }

        if (this.flags.length > this.mineNum) {
          this.flagColor = 'red';
        } else {
          this.flagColor = 'black';
        }

        return;
      }

      if (this.mode == 'sweep') {

        if (!this.flags.includes(index)) {
          this.clickCounter++;
        } else {
          return;
        }

        if(this.hints.includes(index)) {
          tiles[index].style.backgroundColor = "initial";
        }
      }

      this.mineCheck2(index);
    },

    mineCheck2(index) {

    var tiles = document.getElementsByClassName("tileItem");

      if(this.flags.includes(index)) {
        var remove = this.flags.indexOf(index);
        this.flags.splice(remove, 1);
        tiles[index].innerText = '';
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

      if (this.mines.includes(index)) {

        if (this.loopCounter == 0) {
          this.mines = [];
          this.genMines();
          this.minesAround = 0;
          this.mineCheck2(index);
          return;
        } else {
          this.clickCounter--;
          this.gameOver();
          return;
        }
      }

      if (this.loopCounter == 0) {
        if (this.minesAround != 0) {
          this.mines = []
          this.genMines();
          this.minesAround = 0;
          this.mineCheck2(index);
          return;
        }
      }

      if(this.minesAround != 0) {
        tiles[index].innerText = `${this.minesAround}`;
      }

      if (this.minesAround == 0) {
        this.loopCounter++;
        tiles[index].style.backgroundColor = "#CCC"

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

      if (this.mineNum == this.tileNum - this.checkedTiles.length) {
        this.win();
      }
    },

    gameOver() {
      var tiles = document.querySelectorAll(".tileItem");

      tiles.forEach((tile, index) => {
        if(this.mines.includes(index)) {
          tile.innerText = '💣';
        }
      });
      this.showModalBomb = true;
    },

    win() {
      this.showModalWin = true;
    }
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

  span {
    text-align: center;
    font-size: 22px;
    margin-bottom: 20px;
  }

  .difficulty {
    display: flex;
    justify-content: center;

    button {
      margin: 10px;
      font-size: 16px;
    }
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
    margin-bottom: 30px;
    cursor: pointer;

    .tileItem {
      text-align: center;
      line-height: 25px;
      font-size: 17px;
      font-weight: bold;
      background-color: var(--bgColor);
    }
  }

  .modes {
    display: flex;
    justify-content: space-aroun;
    margin-bottom: 20px;

    button {
      flex-grow: 1;
      margin: 0 15px;
    }
  }

  .hintsLeft, .flagsLeft {
    text-align: center;
    font-size: 18px;
    margin-bottom: 10px;
  }

  .flagsLeft span {
    color: var(--color);
  }
}

</style>