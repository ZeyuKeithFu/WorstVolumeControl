<script>
import RandomNumber from "./controllers/RandomNumber.vue"
import OptionList from "./controllers/OptionList.vue"
import Slider from "./controllers/Slider.vue"
import Launcher from "./controllers/Launcher.vue"
import SpinnerWheel from "./controllers/SpinnerWheel.vue"

export default {
    components: {
        RandomNumber,
        OptionList,
        Slider,
        Launcher,
        SpinnerWheel
    },

    data() {
        return {
            volume: 0
        }
    },
    
    mounted() {
        let audio = document.getElementById("audio")
        this.volume = audio.volume
        audio.onvolumechange = (e) => {
            let currentVolume = e.target.volume
            this.volume = currentVolume
        }
    }
}
</script>

<template>
    <div class="player">
        <audio id="audio" controls>
            <source src="../assets/Canon.mp3">
        </audio>
    </div>
    <div class="controller">
        <RandomNumber :volume="volume" />
        <Slider :volume="volume"/>
        <Launcher />
        <SpinnerWheel />
        <OptionList :volume="volume"/>
    </div>
</template>

<style>
.player {
    padding: 20px;
    text-align: center;
}
.divider {
    height: 1px;
    width: 50%;
    background: grey;
}
</style>