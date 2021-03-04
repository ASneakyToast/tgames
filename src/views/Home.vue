<template>
  <div class="home hex-grid" id="hex-grid">
    <section class="hex-grid__list">
      <div v-for="(col, x) in tileArr" :key="x" class="hex-grid__item">
        <div v-for="(row, y) in col" :key="x-y" class="hex-grid__item">
          <div @click="tileClick(x, y)" 
               class="hex-grid__content"
               :class="[ tileArr[x][y].type ]">
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
function getRandom(max) {
  return Math.floor(Math.random() * Math.floor(max));
}
class Zone {
  constructor(base) {
    this.base = base;
    this.x = base.x;
    this.y = base.y;
    this.group = base.group;
    this.tiles = [];
  }

  addAll() {
   alert("whoops"); 
  }
}
let zone = new Zone({x:1,y:3,type:2,group:2});
console.log(zone);

class Tile {
  constructor(x, y, type, group) {
    this.x = x;
    this.y = y;
    this.group = group;
    if (!isNaN(type)) {
      switch (Number(type)) {
        case 0:
          this.type = "water";
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

export default {
  name: 'Home',
  data() {
    return {
      selected: Object,
      colAmount: 4,
      rowAmount: 10,
      tileArr: []
    }
  },
  methods: {
    tileClick(col, row) {
      alert("test!"+col+row);
    }
  },
  created() {
    for (let x=0; x<this.colAmount; x++) {
      this.tileArr.push([]);
      for (let y=0; y<this.rowAmount; y++) {
        this.tileArr[x].push(new Tile(x, y, "1", getRandom(6)));
      }
    }
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

.grass {
  background-color: green;
}

.empty {
  background-color: white;
}

$block: '.hex-grid';

@mixin grid-item($amount) {
    @for $i from 1 through $amount {
        &:nth-of-type(#{$amount}n + #{$i}) {
            grid-column: #{$i + $i - 1} / span 3;
            @if $i % 2 == 0 {
                grid-row: calc(var(--counter) + var(--counter) - 1) / span 2;
            }
        }
    }

    @for $i from 1 through 20 {
        &:nth-of-type(n + #{$i * $amount + 1}) {
            --counter: #{$i + 1};
        }
    }
}

#{$block} {

    &__list {
      --amount: 5;
      position: relative;
      padding: 0;
      margin: 0;
      list-style-type: none;
      display: grid;
      grid-template-columns: repeat(var(--amount), 1fr 2fr) 1fr;
      grid-gap: 2.5rem 5rem;
    }

    &__item {
      position: relative;
      grid-column: 1 / span 3;
      grid-row: calc(var(--counter) + var(--counter)) / span 2;
      filter: drop-shadow(0 0 10px rgba(#444, .08));
      height: 0;
      padding-bottom: 90%;
    }

    &__content {
      position: absolute;
      height: 100%;
      width: 100%;
      font-size: 1.125rem;
      color: #111111;
      clip-path: polygon(75% 0, 100% 50%, 75% 100%, 25% 100%, 0 50%, 25% 0);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      padding: 0;
      margin: 0;
      text-decoration: none;
      text-align: center;
      transition: transform .24s ease-out;
    }
}

@media screen and (min-width: 1440px) {
	#{$block} {
		&__list {
      --amount: 5;
			--counter: 1;
		}

		&__item {
			@include grid-item(5);
		}
	}
}

@media screen and (min-width: 1120px) and (max-width: 1439px) {
	#{$block} {
		&__list {
      --amount: 4;
			--counter: 1;
		}

		&__item {
			@include grid-item(4);
		}
	}
}

@media screen and (min-width: 840px) and (max-width: 1119px) {
	#{$block} {
		&__list {
      --amount: 4;
			--counter: 4;
			grid-gap: 1.5rem 3rem;
		}

		&__item {
			@include grid-item(4);
		}
	}
}

@media screen and (min-width: 480px) and (max-width: 839px) {
	#{$block} {
		&__list {
			--amount: 2;
			--counter: 1;
			grid-gap: 1.5rem 3rem;
		}

		&__item {
			@include grid-item(2);
		}
	}
}

@media screen and (max-width: 479px) {
	#{$block} {
		&__list {
      --amount: 4;
			--counter: 1;
			grid-gap: 0;
		}

		&__item {
			@include grid-item(4);
		}
	}
}

</style>
