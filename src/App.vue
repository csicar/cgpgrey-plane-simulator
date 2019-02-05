<template>
  <div id="app">
    <div class="form">
      <h3 class="heading">Choose a Boarding System: </h3>
      <select v-model="mode">
        <option :value="'random'">Random</option>
        <option :value="'groups'">Groups</option>
        <option :value="'wmi'">Window, Middle, Ile</option>
        <option :value="'inorder'">Front to Back</option>
        <option :value="'perfect'">Perfect 🎉</option>
        <option :value="'custom'">Custom (open Dev Console)</option>
      </select>
      <button @click="restartGame">Run</button>
    </div>
    <div class="container">
      <img src="./assets/plane.png">
      <div 
        class="person"
        :class="{sitting:person.sitting, unloading:person.unloading}"
        :style="position(person)"
        v-for="(person, i) in persons"
        :key="`sit${person.id}`">
      </div>
      <div class="time">{{time}}</div>

    </div>
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'
import { sortBy, reverse } from 'lodash';
const ileY = 417;

function shuffle(array) {
    let counter = array.length;

    // While there are elements in the array
    while (counter > 0) {
        // Pick a random index
        let index = Math.floor(Math.random() * counter);

        // Decrease counter by 1
        counter--;

        // And swap the last element with it
        let temp = array[counter];
        array[counter] = array[index];
        array[index] = temp;
    }

    return array;
}

export default {
  name: 'app',
  data() {
    return {
      time: 0,
      persons: [],
      sortFunction: () => 0,
      mode: "random",
    }
  },
  created() {
    this.createPersons()
      this.startGameLoop();
    console.log("set window.sortFunction = function(personA, personB) { return personA.row - personB.row } ")
    console.log("person hat Properties .row, .column. Your Function should return a Number. Persons with smaller Numbers come first")
  },
  computed: {
    walkingPersons() {
      return this.persons.filter(it => !it.sitting)
    },
    sittingPersons() {
      return this.persons.filter(it => it.sitting)
    }
  },
  methods: {
    position(person) {
      return `top: ${this.topDistForColumn(person.y)}px; left: ${this.leftDistForRow(person.x)}px`;
    },
    leftDistForRow(row) {
      return 220+row*100;
    },
    topDistForColumn(column) {
      if (column === null) return ileY;
      //if (column <= 3) return ileY-(column+1)*70;
      return ileY+3*70-(column + (column >= 3 ? 1 : 0))*70
    },
    createPersons() {
      let persons = [];
      let id = 0;
      for (let x = 0; x < 16; x++) {
        for (let y = 0; y < 6; y++) {
          persons.push({ id:++id, row:x, column:y, x:-id, y:null, sitting: false, unloading: false,
            trySit() {
              if (this.x == this.row) {
                if (!this.unloading) {

                  this.unloading = true;
                  return;
                }
                this.sitting = true;
                this.y = this.column;
              }
            },
            tryMove(persons) {
              if (this.sitting) return;
              if (this.unloading) return;
              const nextPerson = persons.filter(it => !it.sitting).find(it => it.x == this.x+1)
              if (!nextPerson) {
                this.x++;
                return;
              }
            } 
          })
        }
      }
      persons = shuffle(persons);
      persons = persons.sort(this.sortFunction)
      persons.forEach((it, i) => it.x=-i);
      this.persons = persons
    },
    startGameLoop() {
      const that = this;
      
      const gameLoop = () => {
        const sorted = reverse(sortBy(that.persons.filter(it => !it.sitting), 'x'))
        sorted.forEach((person, i) => {
          person.trySit()
          person.tryMove(that.persons)
        })
        that.time++
      }
      function loop() {
        if (that.walkingPersons.length != 0) {
          gameLoop()
          setTimeout(loop, 1000)
        }
      };
      loop();
    },
    restartGame() {
      function byValue(f) {
        return (a, b) => f(a) - f(b)
      } 
      const functions = {
        "inorder": (a, b) => a.id - b.id,
        "groups": byValue((it) => {
          if (it.row > 15) return 0;
          if (it.row > 10) return 1;
          return 2;
        }),
        "random": (it) => 0,
        "wmi": (it) => {
          if (it.column == 0 || it.column == 5) return 0;
          if (it.column == 1 || it.column == 4) return 1;
          return 2;
        },
        "custom": (a, b) => window.sortFunction(a, b),
        "perfect": (a, b) => {
          const remA = a.row%2;
          const remB = b.row%2;
          if (remA == remB) {
            if (a.column == b.column) {
              return b.row-a.row;
            } else {
              if (a.column < 3 && b.column < 3) {
                return a.column-b.column
              } else if (a.column >= 3 && b.column >= 3) {
                return b.column-a.column;
              }
              return a.column-b.column;
            }
          } else {
            return remB-remA;
          }
        }
      }
      this.sortFunction = functions[this.mode]
      this.createPersons();
      console.log("sorted:")
      console.table(JSON.parse(JSON.stringify(this.persons)))
    
    }
  }
}
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
}
.container {
  position: relative;
  flex: 1;
  align-self: center;
  overflow: hidden;
}
.form {
  display: flex;
  justify-content: center;
}
.time {
  position: absolute;
  font-size: 50px;
  left: 50%;
  bottom: 30px;
  margin-left: -60px;
}
.person {
  transition: all 1s;
  position: absolute;
  height: 67px;
  width: 100px;
  background-image: url('./assets/person-walking.png');
  background-repeat: no-repeat;
}
.person.sitting {
  background-image: url('./assets/person-sitting.png');
  background-position-x: 35px; 
  transform: rotate(0deg);

}
.unloading {
  transform: rotate(-30deg);
}
html, body {
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

@media only screen and (max-width: 400px) {
  .container {
    transform: scale(0.2) translateY(-1631px);
  }
}
@media only screen and (min-width: 401px) and (max-width: 600px) {
  .container {
    transform: scale(0.3) translateY(-631px);
  }
}
@media only screen and (min-width: 601px) and (max-width: 950px) {
  .container {
    transform: scale(0.4) translateY(-631px);
  }
}
</style>
