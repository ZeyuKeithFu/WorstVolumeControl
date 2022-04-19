<script>
export default {
    props: {
        volume: Number
    },

    methods: {
        updateVolume() {
            let audio = document.getElementById("audio")
            audio.volume = Math.random()
        }
    }
}
</script>

<template>
    <div class="control-container">
        <div class="control-title">
            <span>Current Volumn Level: </span>
            <span id="volume">{{ Math.round(this.volume * 100) }}%</span>
        </div>
        <button @click="updateVolume">Change</button>
    </div>
</template>

<style scoped>
.control-container {
    text-align: center;
}
.control-title {
    padding: 20px;
    font-size: 24px;
    font-weight: 500;
}
</style>