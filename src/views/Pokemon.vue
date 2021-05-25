<template>
  <div id="game">
    <div class="grid" id="grid">
      <section class="grid-list">
        <div v-for="(col, x) in board.tiles" :key="x" class="grid-item">
          <div v-for="(row, y) in col" :key="x-y" class="grid-item">
            <div @click="useTool(board.tiles[x][y])" 
                 class="grid-content"
                 :class="[ board.tiles[x][y].type ]">

              <!--
              {{ x }},{{ y }} <br>
              {{ board.tiles[x][y].distance }} <br>
              -->

              <!-- this way for iterating through specific known array -->
              <ul v-for="(player, i) in players" :key="i" class="players">
                <li v-if="player.x == x && player.y == y"
                    class="player">
                  <p>{{ player.name }}</p>
                </li>
              </ul>

              <!-- this way for checking what the tile has
              <div v-if="board.tiles[x][y].contains.length">
                <div v-if="board.tiles[x][y].contains.length">
                  <div v-for="(piece, i) in board.tiles[x][y].contains" :key="i" class="pieces">
                    <p>{{ piece.name }}</p>
                  </div>
                </div>
              </div>
              -->
            </div>
          </div>
        </div>
      </section>
    </div>

    <section id="selected">
      <p>Selected: </p>
      <p>{{ selected }}</p>
    </section>
  </div>
</template>

<script>
function getRandom(max) {
  return Math.floor(Math.random() * Math.floor(max));
}

let type = {
  normal: "normal",
  grass: "grass",
  water: "water"
};

class PokeAttack {
  constructor(name, cost, damage, effect) {
    this.name = name;
    this.cost = cost;
    this.damage = damage;
    this.effect = effect;
  }
}

class PokeAttackCreator {
  constructor() {
    let attacks = {};
  }

  create(name, cost, damage, effect) {
    let attack = this.attacks[name];
    if (attack) return attack;

    this.attacks[name] = new PokeAttack(name, cost, damage, effect);
    return this.attacks[name];
  }
}

let attackCreator = new PokeAttackCreator();
const QuickDraw = attackCreator.create("Quick Draw",
  {type["normal"]: 1}, 0, effectCommand("DrawACard"));
const Ram = attackCreator.create("Ram",
  {type["normal"]: 2}, 20, effectCommand("None"));

class AbstractPokemon {
  constructor(name, type, hp, moves) {
    this.name = name;
    this.type = type;
    this.maxHp = hp;
    this.hp = hp;
    this.moves = moves;
  }
}

class Pokemon extends AbstractPokemon {
  constructor(name, type, hp, moves) {
    super(name, type, hp, moves);
    this.hp = this.maxHp;
  }
}

// is this good for another Porygon class?
let Porygon = new Pokemon("Porygon",
  type["normal"], 60,
  [QuickDraw, Ram]);
  


class Tile {
  constructor(x, y, type, group) {
    this.x = x;
    this.y = y;
    this.distance = 0;
    this.contains = [];
    this.group = group;
    //this.isTraversable = true;
    if (!isNaN(type)) {
      switch (Number(type)) {
        case 0:
          this.type = "water";
          //this.isTraversable = false;
          break;
        case 1:
          this.type = "grass";
          break;
        case 2:
          this.type = "tree";
          break;
      }
    } else {
      this.type = type;
    }
  }
}

//TODO: Make a piece class for board items and a card class for non-physical items
// do i mean character instead of piece? that might make more sense
class Piece {
  constructor(tile) {
    this.x = tile.x;
    this.y = tile.y;
    this.type = "default";

    this.maxMoves = 0;
    this.remainingMoves = 0;
  }

  move(to) {
    this.x = to.x;
    this.y = to.y;
  }

  resetMoves() {
    this.remainingMoves = this.maxMoves;
  }
}

class Player extends Piece {
  constructor(tile, id) {
    super(tile); 
    this.type = "player";
    this.id = id;

    this.name = this.type +" "+ this.id;

    this.maxMoves = 3;
  }
}

//TODO: Seperate out hamr logic into another game class
class Board {
  constructor(width, height) {
    this.width = width;
    this.height = height;
    this.tiles = [];
    this.generate();
  }

  generate() {
    this.tiles = [];
    for (let x=0; x<this.width; x++) {
      this.tiles.push([]);
      for (let y=0; y<this.height; y++) {
        this.tiles[x].push(new Tile(x, y, getRandom(2), getRandom(6)));
      }
    }
  }

  // Do i need these functions within the board class?
  remove(piece) {
    alert("removing piece");
    piece.x = -1;
    piece.y = -1;
    return piece;
  }

  place(piece, tile) {
    //if piece has location
    piece.x = tile.x;
    piece.y = tile.y;
  }

  move(piece, to) {
    //this.remove(piece);
    this.place(piece, to);
  }

  getSafeSpawn() {
    let spawnable = [];
    for (let col in this.tiles) {
      for (let row in this.tiles[col]) {
        if (this.isTravesable(this.tiles[col][row])) {
          spawnable.push(this.tiles[col][row]);
        }
      }
    }
    return spawnable[getRandom(spawnable.length)];
  }

  getTileDistance(from, to) {
    this.resetDistances();
    let queue = [from];
    let visited = [];

    while(queue.length) {
      let currentTile = queue.shift();
      visited.push(currentTile);

      if (currentTile == to) {
        return currentTile.distance;
      }

      let surroundings = this.getTileSurroundings(currentTile);
      surroundings.forEach((tile) => {
        if (!visited.includes(tile)) {
          this.tiles[tile.x][tile.y].distance = ( currentTile.distance+1 );

          //TODO: for shortest path line
          this.tiles[tile.x][tile.y].parent = this.tiles[currentTile.x][currentTile.y];

          queue.push(this.tiles[tile.x][tile.y]);
        }
      });
    }
  }
      
  updateDistances(centerTile) {
    this.resetDistances();
    let queue = [centerTile];
    let visited = [];
    
    while(queue.length) {
      let currentTile = queue.shift();
      visited.push(currentTile);
      
      let surroundings = this.getTileSurroundings(currentTile); 
      surroundings.forEach((tile) => {
        if (!visited.includes(tile) && this.isTravesable(tile)) {
          this.tiles[tile.x][tile.y].distance = ( currentTile.distance+1 );
          queue.push(this.tiles[tile.x][tile.y]);
        }
      });
    }
  }

  isTravesable(tile) {
    if (tile.type == "water") {
      return false;
    } else {
      return true;
    }
  }

  resetDistances() {
    for (let col in this.tiles) {
      for (let row in this.tiles[col]) {
        let tile = this.tiles[col][row]
        tile.distance = 0;
      }
    }
  }

  getTile(x, y) {
    return this.tiles[x][y];
  }

  getTileSurroundings(tile) {
    let surroundings = [];
    // top, right, bottom, left
    if (( tile.y - 1 ) >= 0) {
      surroundings.push(this.tiles[tile.x][tile.y - 1]);
    }
    if (( tile.x + 1 ) < this.width) {
      surroundings.push(this.tiles[tile.x + 1][tile.y]);
    }
    if (( tile.y + 1 ) < this.height) {
      surroundings.push(this.tiles[tile.x][tile.y + 1]);
    }
    if (( tile.x - 1 ) >= 0) {
      surroundings.push(this.tiles[tile.x - 1][tile.y]);
    }
    return(surroundings);
  }
}


// ugh, another class, do i need this?
/*
class Game {
  contructor(board, pieces) {
    this.board = board;
    this.pieces = pieces;
  }
}
*/

export default {
  name: 'Home',
  data() {
    return {
      players: Array,
      board: Object,
      cols: 5,
      rows: 5,
      selected: false,
      currentPlayer: Object,

      tool: "debug-move"
    }
  },
  methods: {
    tileClick(col, row) {
      alert("test!"+col+row);
    },
    checkDistance(tile) {
      if (!this.selected) {
        this.selected = tile;
      } else {
        this.board.getTileDistance(this.selected, tile);
        this.selected = false;
      }
    },
    debugDistance(tile) {
      this.selected = tile;
      this.board.updateDistances(tile);
    },
    debugMove(target) {
      if (!this.selected) {
        this.selected = target;
        alert("selected");
      } else {
        alert("moving");
        alert(this.selected.x +","+ this.selected.y);
        this.currentPlayer.move(this.selected, target);
      }
    },
    useTool(tile) {
      if (this.tool == "debug-surroundings") {
        this.checkSurroundings(tile);
      } else if (this.tool == "debug-distance") {
        this.debugDistance("reset");
        this.debugDistance(tile);
      } else if (this.tool == "check-distance") {
        this.checkDistance(tile);
      } else if (this.tool == "debug-move") {
        alert("use tool");
        this.debugMove(tile);
      }
    },
  },
  created() {
    this.board = new Board(this.cols, this.rows);
    //TODO: Create onGenerate hook for custom logic, then move class to its own module.
    this.board.generate();

    //Place Players + items
    let safeSpawn = this.board.getSafeSpawn();
    let playerOne = new Player(safeSpawn, 1);
    this.players = [playerOne];
    this.board.place(playerOne, safeSpawn);
    this.selectedPlayer
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

#game {
  display: grid;
  grid-template-rows: 1fr 150px;
}

ul, li, p, div {
  margin: 0;
  padding: 0;
}

.grass {
  background-color: green;
}

.water {
  background-color: blue;
}

.empty {
  background-color: white;
}

.grid-list {
  --amount: 25;
  --cols: 5;
  display: grid;
  grid-wrap: wrap;
  grid-template-columns: repeat(var(--cols), 1fr);

  position: relative;
  padding: 0;
  margin: 0;
}

.grid-item {
  height: 0;
  padding-bottom: 90%;
  position: relative;
}

.grid-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  position: absolute;
  height: 100%;
  width: 100%;
  color: #111111;
  padding: 0;
  margin: 0;
}

</style>
