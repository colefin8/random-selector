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
        <div v-else :class="[{ spin: spinning }, 'spinner']">
          <div
            class="name"
            :class="{ 'name--small': name.length > 17 }"
            v-for="(name, index) in shuffledNames"
            :key="name + index"
          >
            {{ name.toUpperCase() }}
          </div>
        </div>
      </Transition>
    </div>
    <div id="lights"></div>
    <div id="button" :class="{ clicked: clicked }" @click.stop="click"></div>
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
      this.shuffledNames = this.shuffle(this.names);
      this.spin();
      setTimeout(() => {
        this.clicked = false;
      }, 1000);
    },
    spin() {
      function getOccurrence(array) {
        var count = {};
        for (let i = 0; i < 100; i++) {
          array.forEach((v) => {
            if (parseInt(v) === i + 1) {
              (i + 1).toString() in count ? count[i + 1] +=1 : count[i + 1] = 1
            }
          });
        }
        return count;
      }
      console.log(getOccurrence(this.shuffledNames));
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
          this.names.splice(
            this.names.findIndex((e) => e === this.winner),
            1
          );
        }
      }, 9000);
    },
    shuffle(array) {
      let multArr = [];
      for (let i = 0; i < 100; i++) {
        let randomIndex = Math.floor(Math.random() * array.length);
        multArr.push(array[randomIndex]);
      }
      return multArr;
    },
  },
  created() {
    this.jsConfetti = new JSConfetti();
  },
};
</script>

<style scoped>
.wrapper {
  position: relative;
  height: 100%;
  margin-top: calc(50vh - (730.5px / 2));
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
