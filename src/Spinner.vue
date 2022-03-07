<template>
    <div class="wrapper">
        <img class="roulette" src="./assets/roulette_machine.svg"/>
        <div id="text">
            <div v-if="!spinning && !result" class="name" style="display: flex; justify-content: center; align-items: center;"><img src="./assets/awardco-horizontal-black 1.svg"/></div>
            <div v-else :class="[{'spin': spinning}, 'spinner']">
                <div class="name" v-for="(name, index) in shuffledNames" :key="name + index">{{ name.toUpperCase() }}</div>
            </div>
        </div>
        <div id="lights"></div>
        <div id="button" :class="{'clicked': clicked}" @click.stop="click"></div>
    </div>
</template>

<script>
export default {
    name: 'spinner',
    props: {
        names: {
            type: Array,
            required: true,
        }
    },
    data() {
        return {
            clicked: false,
            spinning: false,
            result: false,
            shuffledNames: [...this.names]
        }
    },
    methods: {
        click() {
            this.spinning = false;
            this.clicked = true;
            this.shuffledNames = this.shuffle(this.names)
            this.spin()
            setTimeout(()=> {
                this.clicked = false;
            },1000)
        },
        spin() {
            setTimeout(() => {
                this.spinning = true;
            }, 500);
            this.result = true;

        },
        shuffle(array) {
            let currentIndex = array.length,  randomIndex;
            while (currentIndex != 0) {
                randomIndex = Math.floor(Math.random() * currentIndex);
                currentIndex--;
                [array[currentIndex], array[randomIndex]] = [
                array[randomIndex], array[currentIndex]];
            }
            let multArr = []
            if(array.length < 50 && array.length > 0){
                while(multArr.length < 100){
                    multArr = [...multArr, ...array]
                }
            } else if (array.length >= 50){
                multArr = array.slice(0,50)
            } else multArr = [...array]
            return multArr;
        },
    }
}
</script>

<style scoped>
    .wrapper {
        position: relative;
        height: 100%;
        margin-top: calc(50vh - (730.5px / 2))
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
    }
    .spinner {
        position: absolute;
        width: 100%;
    }
    #lights {
        left: 0;
        right: 0;
        margin: 0px auto;
        position: absolute;
        height: 652.5px;
        width: 730.5px;
        background-image: url('./assets/lights_spritesheet.png');
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
        background-image: url('./assets/button_spritesheet.png');
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
        animation: press .5s steps(5) 1;
    }
    .spin {
        animation: spin 12s ease-in-out;
        animation-fill-mode: forwards;
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