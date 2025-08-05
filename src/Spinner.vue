<template>
  <div class="wrapper">
    <img class="roulette" src="./assets/roulette_machine.svg" />
    <div id="text">
      <Transition>
        <div
          v-if="!spinning && !result"
          class="name"
          style="display: flex; justify-content: center; align-items: center"
        >
          <img src="./assets/awardco-horizontal-black 1.svg" />
        </div>
        <div v-else :class="['spinner', { spin: spinning }]">
          <div
            class="name"
            :class="{ 'name--small': name && name.toString().length > 17 }"
            v-for="(name, index) in shuffledNames"
            :key="name + index"
          >
            {{ name && name.toString().toUpperCase() }}
          </div>
        </div>
      </Transition>
    </div>
    <div id="lights"></div>
    <div
      id="button"
      :class="{ clicked: clicked }"
      @click.stop="click"
      title="Click to spin!"
    ></div>
  </div>
</template>

<script>
import JSConfetti from "js-confetti";

export default {
  name: "spinner",
  props: {
    deleteWinner: {
      type: Boolean,
      required: false,
      default: true,
    },
    names: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      clicked: false,
      spinning: false,
      result: false,
      shuffledNames: [...this.names],
      winner: "",
      jsConfetti: null,
      finished: true,
    };
  },
  methods: {
    click() {
      if (!this.finished) return;
      this.spinning = false;
      this.finished = false;
      this.clicked = true;
      const winnerIndex = Math.floor(Math.random() * this.names.length);
      this.winner = this.names[winnerIndex];
      this.shuffledNames = this.createRouletteList(this.names, this.winner);

      this.spin();
      setTimeout(() => {
        this.clicked = false;
      }, 1000);
    },
    spin() {
      setTimeout(() => {
        this.spinning = true;
      }, 500);
      this.result = true;
      setTimeout(() => {
        this.finished = true;
        this.jsConfetti.addConfetti({
          confettiColors: ["#FA5959", "#20C4F4", "#FDAF08", "#44DDB4"],
        });

        if (this.deleteWinner === true) {
          const winnerIndex = this.names.findIndex((e) => e === this.winner);
          if (winnerIndex !== -1) {
            this.names.splice(winnerIndex, 1);
          }
        }
        this.$emit('winner-selected', this.winner);
      }, 9000);
    },
    createRouletteList(array, winner) {

      let rouletteList = [];
      const totalDisplayNames = 99;
      let namePool = [];
      array.forEach(name => {
        const repeatCount = Math.floor(Math.random() * 2) + 2;
        for (let i = 0; i < repeatCount; i++) {
          namePool.push(name);
        }
      });
      namePool = this.shuffleArray(namePool);
      const middleInsertPoint = Math.floor(Math.random() * 40) + 30;
      for (let i = 0; i < totalDisplayNames; i++) {
        if (i === middleInsertPoint) {
          rouletteList.push(winner);
        }
        const randomIndex = Math.floor(Math.random() * namePool.length);
        rouletteList.push(namePool[randomIndex]);
      }
      rouletteList.push(winner);

      console.table(rouletteList)
      return rouletteList;
    },
    shuffleArray(array) {
      const newArray = [...array];
      for (let i = newArray.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [newArray[i], newArray[j]] = [newArray[j], newArray[i]];
      }
      return newArray;
    },
  },
  created() {
    this.jsConfetti = new JSConfetti();

    this.handleKeyDown = (e) => {
      if ((e.key === " " || e.key === "Enter") && this.finished) {
        this.click();
      }
    };
    window.addEventListener("keydown", this.handleKeyDown);
    this.updateMachineScale = () => {
      const viewportHeight = window.innerHeight;
      const viewportWidth = window.innerWidth;
      const originalHeight = 730.5;
      const originalWidth = 912;
      const heightScale = (viewportHeight * 0.85) / originalHeight;
      const widthScale = (viewportWidth * 0.9) / originalWidth;
      const scale = Math.min(heightScale, widthScale, 1);
      document.documentElement.style.setProperty("--machine-scale", scale);
    };
    this.updateMachineScale();
    window.addEventListener("resize", this.updateMachineScale);
  },

  beforeUnmount() {
    window.removeEventListener("keydown", this.handleKeyDown);
    window.removeEventListener("resize", this.updateMachineScale);
  },
};
</script>

<style scoped>
:root {
  --machine-scale: 1;
}

.wrapper {
  position: relative;
  width: 912px;
  height: 730.5px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%) scale(var(--machine-scale, 1));
  transform-origin: center center;
  min-width: 300px;
  min-height: 240px;
}
.roulette {
  position: absolute;
  width: 912px;
  left: 0;
  right: 0;
  margin: 0px auto;
}
.name {
  height: 100%;
  width: 100%;
  text-align: center;
  line-height: 117px;
  font-size: 36px;
  font-weight: bold;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  box-sizing: border-box;
}
.name--small {
  font-size: 28px;
}
.spinner {
  position: absolute;
  width: 100%;
  box-sizing: border-box;
  padding: 0px 8px;
}
#lights {
  left: 0;
  right: 0;
  margin: 0px auto;
  position: absolute;
  height: 652.5px;
  width: 730.5px;
  background-image: url("./assets/lights_spritesheet.png");
  background-size: 200%;
  background-position: 0px 0px;
  animation: play 1.5s steps(2) infinite;
}
#button {
  left: 0;
  right: 0;
  top: 655px;
  margin: 0px auto;
  position: absolute;
  height: 98px;
  width: 156px;
  background-size: 500%;
  background-image: url("./assets/button_spritesheet.png");
  cursor: pointer;
  transition: transform 0.2s ease;
}

#button:hover {
  transform: scale(1.05);
  filter: brightness(1.1);
}
#text {
  height: 117px;
  left: 0;
  right: 0;
  top: 343.5px;
  margin: 0px auto;
  position: absolute;
  width: 405px;
  overflow: hidden;
}
.clicked {
  animation: press 0.5s steps(5) 1;
}
.spin {
  animation: spin 9s ease-in-out;
  animation-fill-mode: forwards;
}
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
@keyframes play {
  100% {
    background-position: 200%;
  }
}
@keyframes press {
  100% {
    background-position: 500%;
  }
}
@keyframes spin {
  100% {
    transform: translateY(calc(-100% + 117px));
  }
}
</style>
