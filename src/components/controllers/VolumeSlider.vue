<script>
const FRAME_RATE = 60
const g = 0.2 / FRAME_RATE // simulation of gravitational constant

export default {
    props: {
        volume: Number
    },

    data() {
        return {
            audio: null,
            container: null,
            timer: null,
            isDragging: false,
            centerX: 0, centerY: 0, // slider center position
            u: 0, // current acceleration
            v: 0, // current velocity
            angleOffset: 0, // angle of initial mousedown point
            angle: 0 // current angle
        }
    },

    mounted() {
        this.audio = document.getElementById("audio")
        this.container = document.getElementById("slider-container")
    },

    methods: {
        updateVolume() {
            this.u = Math.sin(this.angle) * g
            this.v += this.u
            let currentVolume = this.volume + this.v
            if (currentVolume > 1 || currentVolume < 0) {
                if (currentVolume > 1) {
                    currentVolume = 1
                } else {
                    currentVolume = 0
                }
                this.v = 0
            }
            this.audio.volume = currentVolume
        },

        onMouseDown(e) {
            this.isDragging = true
            let bounds = this.container.getBoundingClientRect()
            this.centerX = (bounds.left + bounds.right) / 2
            this.centerY = (bounds.top + bounds.bottom) / 2
            this.angleOffset = Math.atan2(
                e.clientY - this.centerY, e.clientX - this.centerX)
            // update in preset frame rate
            this.timer = setInterval(this.updateVolume, 1000 / FRAME_RATE)
        },

        onMouseMove(e) {
            if (this.isDragging) {
                this.angle = Math.atan2(
                    e.clientY - this.centerY, e.clientX - this.centerX) - this.angleOffset
                this.container.style.transform = `rotate(${this.angle * 180 / Math.PI}deg)`
            }
        },

        onMouseLeave() {
            this.isDragging = false
            // reset status
            clearInterval(this.timer)
            this.timer = null
            this.u = 0
            this.v = 0
            this.angle = 0
            // reset style
            this.container.style.transform = "rotate(0)"
        }
    }
}
</script>

<template>
    <div class="control-container"
        @mousemove="onMouseMove($event)"
        @mouseleave="onMouseLeave()"
        @mouseup="onMouseLeave()">
        <h3 class="control-title">Volume: {{ Math.round(this.volume * 100) }}</h3>
        <div id="slider-container"
            :class="this.isDragging ? '' : 'transition'"
            @mousedown="onMouseDown($event)">
            <input id="slider" disabled="true" type="range" :value="this.volume * 100">
        </div>
    </div>
</template>

<style scoped>
.control-container {
    height: 300px;
    display: flex;
    justify-content: center;
    align-items: center;
}
.control-title {
    width: 120px;
}
#slider-container {
    background: #f0f0f0;
    margin-left: 20px;
    padding: 10px 20px;
    border: 1px solid black;
    display: flex;
    cursor: grab;
}
#slider-container.transition {
    transition-timing-function: ease-out;
    transition: transform 0.3s;
}
#slider {
    width: 200px;
}
input[type=range]:disabled::-webkit-slider-thumb {
    box-shadow: 2px 2px 2px #000000, 0px 0px 2px #0d0d0d;
    border-radius: 50%;
}
input[type=range]:disabled::-moz-range-thumb {
    box-shadow: 2px 2px 2px #000000, 0px 0px 2px #0d0d0d;
    border-radius: 50%;
}
input[type=range]:disabled::-ms-thumb {
    box-shadow: 2px 2px 2px #000000, 0px 0px 2px #0d0d0d;
    border-radius: 50%;
}
</style>