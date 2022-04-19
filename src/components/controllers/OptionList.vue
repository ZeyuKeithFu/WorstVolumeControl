<script>
export default {
    props: {
        volume: Number
    },

    methods: {
        checkVolume(v, number) {
            return Math.round(v * 100) == number
        },

        changeVolume(e) {
            if (e.target.checked) {
                let audio = document.getElementById("audio")
                audio.volume = e.target.value / 100
            }
        }
    }
}
</script>

<template>
    <div class="control-container">
        <div>Volume Control</div>
        <div class="control-options">
            <li v-for="n in 100" :key="n">
                <input type="checkbox"
                    :value="n"
                    :checked="checkVolume(this.volume, n)"
                    @change="changeVolume($event)">
                <span>{{ n }}</span>
            </li>
            <br/>
            <li>
                <input type="checkbox"
                    value="0"
                    :checked="checkVolume(this.volume, 0)"
                    @change="changeVolume($event)">
                <span>Muted</span>
            </li>
        </div>
    </div>
</template>

<style scoped>
li {
    display: inline-block;
    padding: 4px;
}
span {
    margin-left: 4px;
}
.control-container {
    width: 33.3%;
    padding: 20px;
    margin: auto;
    margin-top: 20px;
    border: 1px;
    border-style: solid;
    border-color: grey;
}
.control-options {
    margin-top: 10px;
    text-align: center;
}
</style>