<template>
  <div id="plane">
    <select v-model="mode">
      <option :value="'random'">Random</option>
      <option :value="'groups'">Groups</option>
      <option :value="'wmi'">Window, Middle, Ile</option>
    </select>
    <div class="container">
      <img src="./assets/plane.png">
      <div 
        class="person"
        :class="{sitting:person.sitting}"
        :style="position(person)"
        v-for="(person, i) in persons"
        :key="`sit${person.id}`">
      </div>
    </div>
    <div class="time">{{time}}</div>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
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
  name: 'plane',
  data() {
    const persons = [];
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
    shuffle(persons);
    persons.forEach((it, i) => it.x=-i);
    return {
      time: 0,
      persons: shuffle(persons),
      mode: "Random"
    }
  },
  created() {
    const that = this;
      <option>Groups</option>
    function findPersonInfront(person, persons) {
      const ret =  that.persons
        .filter(it => !it.sitting)
        .find(it => {
          return person.x - it.x < 2
          })
      return ret;
    }
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
      if (column >= 3) return ileY+(column-2)*70;
      return ileY-(column+1)*70
    }
  },
  watch: {
    mode(val) {
      console.log("new mode", val)
    }
  }
}
</script>

<style>
#plane {
  display: flex;
  flex-direction: column;
}
.container {
  position: relative;
  flex: 1;
  align-self: center;
}
.time {
  position: absolute;
  font-size: 50px;
  left: 50%;
  bottom: -420px;
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
  
}
html, body {
  margin: 0;
  padding: 0;
}
</style>
