<script>
import RandomNumber from "./controllers/RandomNumber.vue"
import OptionList from "./controllers/OptionList.vue"
import VolumeSlider from "./controllers/VolumeSlider.vue"
import VolumeLauncher from "./controllers/VolumeLauncher.vue"
import SpinnerWheel from "./controllers/SpinnerWheel.vue"

export default {
    components: {
        RandomNumber,
        OptionList,
        VolumeSlider,
        VolumeLauncher,
        SpinnerWheel
    },

    data() {
        return {
            volume: 0,
            controllers: [
                {
                    "item": "RandomNumber",
                    "hint": ""
                },
                {
                    "item": "VolumeSlider",
                    "hint": "Grab and tilt the slider"
                },
                {
                    "item": "VolumeLauncher",
                    "hint": "Press the icon and release to launch the volume indicator"
                },
                {
                    "item": "SpinnerWheel",
                    "hint": ""
                },
                {
                    "item": "OptionList",
                    "hint": ""
                }
            ],
            current: 0 // current controller index
        }
    },
    
    mounted() {
        let audio = document.getElementById("audio")
        this.volume = audio.volume
        audio.onvolumechange = (e) => {
            let currentVolume = e.target.volume
            this.volume = currentVolume
        }
    },

    methods: {
        switchController(isNext) {
            let len = this.controllers.length
            if (isNext) {
                this.current = (this.current + 1) % len
            } else {
                this.current = (this.current - 1 + len) % len
            }
        }
    }
}
</script>

<template>
    <div class="title">
        <h1>Canon in D Major</h1>
    </div>
    <div class="player">
        <div class="previous"
            @click="switchController(false)">
            <img :src="'icon/arrow.svg'">
        </div>
        <div class="audio-container">
            <audio id="audio" controls>
                <source :src="'raw/Canon.mp3'">
            </audio>
        </div>
        <div class="next"
            @click="switchController(true)">
            <img :src="'icon/arrow.svg'">
        </div>
    </div>
    <div class="controller">
        <component :is="controllers[current].item" :volume="volume"></component>
    </div>
    <div class="hint">
        <div>{{ controllers[current].hint }}</div>
    </div>
</template>

<style scoped>
.title {
    text-align: center;
}
.player {
    width: 60%;
    margin: auto;
    padding: 20px;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
}
.audio-container {
    display: flex;
    align-content: center;
}
.previous {
    width: 50px;
    height: 50px;
    cursor: pointer;
}
.next {
    width: 50px;
    height: 50px;
    transform: rotate(180deg);
    cursor: pointer;
}
.hint {
    color: #b5b5b5;
    text-align: center;
}
</style>