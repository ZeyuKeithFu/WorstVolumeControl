<script>
const FRAME_RATE = 60
const INITIAL_OMEGA = 0.6 // initial angular velocity of wheel

export default {
    data() {
        return {
            audio: null,
            wheel: null,
            button: null,
            colorSet: ["#f04c32", "#f09e32", "#e3f032", "#4287f5", "#42f56c", "#b14df0"],
            volumeSet: [],
            canSpin: false,
            shrinkAnim: null,
            buttonScale: 1,
            wheelAngle: 0,
            wheelW: INITIAL_OMEGA
        }
    },

    mounted() {
        this.audio = document.getElementById("audio")
        this.wheel = document.getElementById("wheel")
        this.button = document.getElementById("wheel-button")
        this.initVolumeOptions()
    },

    methods: {
        initVolumeOptions() {
            this.volumeSet = []
            for (let i = 0; i < 12; i++) {
                let volume = Math.floor(Math.random() * 100)
                while (this.volumeSet.includes(volume)) {
                    volume = Math.floor(Math.random() * 100)
                }
                this.volumeSet.push(volume)
            }
        },

        onButtonPress() {
            this.shrinkAnim = setInterval(this.shrinkButton, 1000 / FRAME_RATE)
            this.canSpin = true
        },

        onButtonRelease() {
            if (this.canSpin) {
                this.canSpin = false
                clearInterval(this.shrinkAnim)
                this.shrinkAnim = null
                this.buttonScale = 1
                this.button.style.transform = `scale(1)`
                this.spinWheel()
            }
        },

        shrinkButton() {
            this.buttonScale *= 0.99
            if (this.buttonScale < 0.6) {
                this.buttonScale = 0.6
            } else {
                this.wheelW += 0.02
            }
            this.button.style.transform = `scale(${this.buttonScale})`
        },

        spinWheel() {
            let spinAnim = setInterval(() => {
                this.wheelAngle += this.wheelW
                if (this.wheelAngle > 2 * Math.PI) {
                    this.wheelAngle -= 2 * Math.PI
                }
                this.wheelW -= 0.01
                this.wheel.style.transform = `rotate(${this.wheelAngle * 180 / Math.PI}deg)`
                if (this.wheelW < 0) {
                    clearInterval(spinAnim)
                    this.updateVolume()
                    this.wheelW = INITIAL_OMEGA
                    this.wheelAngle = 0
                }
            }, 1000 / FRAME_RATE)
        },

        updateVolume() {
            let step = Math.floor(this.wheelAngle / (Math.PI / 6))
            let offset = this.wheelAngle - (step * Math.PI / 6)
            if (offset > Math.PI / 12) {
                step += 1
            }
            this.audio.volume = this.volumeSet[step] / 100
        }
    }
}
</script>

<template>
    <div class="control-container">
        <div class="title">
            WHEEL OF VOLUME
        </div>
        <div class="wheel-container">
            <div class="indicator"></div>
            <div id="wheel">
                <div v-for="i in 12"
                    :key="i"
                    :style="`
                        width: ${Math.tan(15 / 180 * Math.PI) * 2 * 50}%;
                        background-color: ${this.colorSet[(i - 1) % 6]};
                        transform: rotate(${-(i - 1) * 30}deg)
                    `">
                    <h2>{{ this.volumeSet[i - 1] }}</h2>
                </div>
            </div>
        </div>
        <div id="wheel-button"
            @mousedown="onButtonPress()"
            @mouseup="onButtonRelease()"
            @mouseleave="onButtonRelease()">
            PRESS AND RELEASE
        </div>
    </div>
</template>

<style scoped>
.control-container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}
.title {
    padding: 10px;
    text-transform: uppercase;
    font-family: verdana;
    font-size: 2em;
    font-weight: 700;
    color: gold;
    cursor: default;
    text-shadow: 1px 1px 1px #a8a81c,
        1px 2px 1px #a8a81c,
        1px 3px 1px #a8a81c,
        1px 4px 1px #a8a81c,
        1px 5px 1px #a8a81c,
        1px 6px 1px #a8a81c,
        1px 7px 1px #a8a81c,
        1px 8px 1px #a8a81c,
        1px 9px 1px #a8a81c,
        1px 10px 1px #a8a81c,
        1px 18px 6px rgba(126, 126, 5, 0.4),
        1px 22px 10px rgba(126, 126, 5, 0.2),
        1px 25px 35px rgba(126, 126, 5, 0.2),
        1px 30px 60px rgba(126, 126, 5, 0.4);
}
.wheel-container {
    margin: 20px;
}
.indicator {
    margin: auto;
    width: 20px;
    height: 20px;
    background: black;
    clip-path: polygon(100% 0, 50% 100%, 0 0);
}
#wheel {
    width: 300px;
    height: 300px;
    border-radius: 50%;
    overflow: hidden;
    position: relative;
}
#wheel div {
    height: 50%;
    margin: auto;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    text-align: center;
    vertical-align: middle;
    clip-path: polygon(100% 0, 50% 100%, 0 0);
    transform-origin: 50% 100%;
}
h2 {
    font-family: verdana;
}
#wheel-button {
    font-family: verdana;
    font-weight: 700;
    border-radius: 8px;
    padding: 10px 20px;
    cursor: default;
    background: linear-gradient(
        to bottom,
        #ebcc34,
        #e3e765,
        #ebcc34
    );
}
</style>