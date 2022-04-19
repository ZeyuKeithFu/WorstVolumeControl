<script>
const FRAME_RATE = 60
const TRACK_LENGTH = 200
const ICON_WIDTH = 60
// angular velocity (omega) of the launcher
// time cost is 1s to get to max angle
const O_LAUNCHER = -Math.PI / 4 / (1.0 * FRAME_RATE)
// max landing time in seconds
const MAX_T = 1
// initial velocity when launching
const V_0 = TRACK_LENGTH / Math.cos(Math.PI / 4) / (MAX_T * FRAME_RATE)
// max height when launching
const MAX_H = ICON_WIDTH * Math.sin(Math.PI / 4)
// simulation of gravitational constant
const g = (MAX_H + V_0 * Math.sin(Math.PI / 4) * (MAX_T * FRAME_RATE)) * 2 /
    ((MAX_T * FRAME_RATE) * (MAX_T * FRAME_RATE))

export default {
    data() {
        return {
            audio: null, icon: null, track: null, indicator: null,
            powerAnim: null,
            isPowering: false,
            angle: 0, // current launcher angle
            x: 0, y: 0, // bullet position
            vx: 0, vy: 0, // bullet velocity
            trackW: 0 // width of volume track
        }
    },

    mounted() {
        this.audio = document.getElementById("audio")
        this.icon = document.getElementById("icon")
        this.track = document.getElementById("track")
        this.indicator = document.getElementById("indicator")
    },

    methods: {
        powerUp() {
            this.angle += O_LAUNCHER
            if (this.angle < -(Math.PI / 4)) {
                this.angle = -(Math.PI / 4)
            }
            this.icon.style.transform = `rotate(${this.angle * 180 / Math.PI}deg)`
        },

        launch() {
            this.y = this.angle / (Math.PI / 4) * MAX_H
            this.vx = V_0 * Math.cos(this.angle)
            this.vy = V_0 * Math.sin(this.angle)
            this.indicator.style.visibility = "visible"
            let bulletAnim = setInterval(() => {
                if (this.y > 0) {
                    this.indicator.style.transform =
                        `translateX(${-6 + this.x}px) translateY(${-4}px)`
                    this.audio.volume = Math.floor(this.x) / 200
                    this.x = this.y = this.vx = this.vy = 0
                    clearInterval(bulletAnim)
                } else {
                    this.indicator.style.transform =
                        `translateX(${-6 + this.x}px) translateY(${-4 + this.y}px)`
                }
                this.vy += g
                this.y += this.vy
                this.x += this.vx
            }, 1000 / FRAME_RATE)
        },

        onMouseDown() {
            this.isPowering = true
            this.powerAnim = setInterval(this.powerUp, 1000 / FRAME_RATE)
        },

        onMouseUp() {
            if (this.isPowering) {
                this.isPowering = false
                clearInterval(this.powerAnim)
                this.powerAnim = null
                this.launch()
                this.angle = 0
                // reset style
                this.icon.style.transform = `rotate(0)`
            }
        },
    }
}
</script>

<template>
    <div class="control-container">
        <div id="icon"
            @mousedown="onMouseDown()"
            @mouseup="onMouseUp()"
            @mouseleave="onMouseUp()"
            :class="this.isPowering ? '' : 'transition'"></div>
        <div id="track">
            <div id="indicator"></div>
        </div>
    </div>
</template>

<style scoped>
.control-container {
    padding: 20px;
    display: flex;
    justify-content: center;
    align-items: center;
}
#icon {
    width: 60px;
    height: 60px;
    background-color: lightgrey;
    transform-origin: 10% center;
    mask-image: url("../../assets/volume.svg");
    mask-repeat: no-repeat;
    mask-size: 100%;
    -webkit-mask-image: url("../../assets/volume.svg");
    -webkit-mask-image-repeat: no-repeat;
    -webkit-mask-size: 100%;
}
#icon:hover {
    background-color: rgba(102, 51, 153, 0.3);
}
#icon.transition {
    transition-timing-function: ease-out;
    transition: transform 0.2s;
}
#track {
    width: 200px;
    height: 4px;
    border-radius: 2px;
    background: lightgrey;
}
#indicator {
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background: rgba(102, 51, 153);
    visibility: hidden;
    transform: translateX(-6px) translateY(-4px);
}
</style>