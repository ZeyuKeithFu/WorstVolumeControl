<script>
import * as ort from 'onnxruntime-web'
import html2canvas from 'html2canvas'

const CANVAS_W = 500
const DOT_RADIUS = 10

export default {
    mounted() {
        this.audio = document.getElementById("audio")
        // Generate random dots
        this.dots = Array.from(Array(100), () => new Array(2))
        for (let i = 0; i < 100; i++) {
            this.dots[i][0] = Math.round((CANVAS_W - DOT_RADIUS * 2) * Math.random())
            this.dots[i][1] = Math.round((CANVAS_W - DOT_RADIUS * 2) * Math.random())
        }
    },

    props: {
        volume: Number
    },

    data() {
        return {
            audio: null,
            dots: Array.from(Array(100), () => new Array(2)),
            moving: false, // flag indicating dot is moving
            selecting: false, // flag indicating selecting area
            dotStart: {
                x: 0,
                y: 0
            }, // where mouse click on dot
            dotPosition: {
                left: 0,
                top: 0
            }, // where the dot boundary is located
            selectStart: {
                x: 0,
                y: 0
            },
            movingDotId: ""
        }
    },

    methods: {
        updateVolume(v) {
            this.audio.volume = v
        },

        recognize() {
            let area = document.getElementById("pattern-container")
            let areaBounds = area.getBoundingClientRect()
            let selected = document.getElementById("pattern-selector")
            if (selected.hidden) {
                console.log("Not select yet")
                return
            }
            let params = getComputedStyle(selected)
            // selected area params
            let offsetX = parseInt(params.left) - areaBounds.left
            let offsetY = parseInt(params.top) - areaBounds.top
            let w = parseInt(params.width)
            let h = parseInt(params.height)
            if (area && w > 0 && h > 0) {
                // transform selected area to canvas
                html2canvas(area, {
                    x: offsetX,
                    y: offsetY,
                    width: w,
                    height: h
                }).then(canvas => {
                    let resizedImageData = this.processImage(canvas, 28)
                    let inputTensor = this.imageDataToTensor(resizedImageData, [1, 1, 28, 28])
                    this.run(inputTensor)
                })
            } else {
                console.log("Selected area: ", w, h)
            }
        },

        processImage(input, width) {
            let canvas = document.createElement("canvas")
            let ctx = canvas.getContext("2d")

            // resize image in a width * width box
            if (input.width > input.height) {
                canvas.width = width
                canvas.height = canvas.width * (input.height / input.width)
            } else {
                canvas.height = width
                canvas.width = canvas.height * (input.width / input.height)
            }

            // draw scaled image
            ctx.drawImage(input, 0, 0, canvas.width, canvas.height);

            // used for debugging
            // document.getElementById("scaled-img").src = canvas.toDataURL();

            // return data
            return ctx.getImageData(0, 0, width, width).data;
        },

        imageDataToTensor(data, dims) {
            // 1a. Extract the R, G, and B channels from the data
            // To optimize: for MNIST model, greyscale should be enough for work
            const [R, G, B] = [[], [], []]
            for (let i = 0; i < data.length; i += 4) {
                R.push(data[i]);
                G.push(data[i + 1]);
                B.push(data[i + 2]);
                // 2. skip data[i + 3] thus filtering out the alpha channel
            }
            // 1b. concatenate RGB ~= transpose [28, 28, 1] -> [1, 28, 28]
            const transposedData = R.concat(G).concat(B);

            // 3. convert to float32
            let i, l = transposedData.length; // length, we need this for the loop
            const float32Data = new Float32Array(1 * 28 * 28); // create the Float32Array for output
            for (i = 0; i < l; i++) {
                float32Data[i] = transposedData[i] / 255.0; // convert to float
            }

            const inputTensor = new ort.Tensor("float32", float32Data, dims);
            return inputTensor;
        },

        // Model prediction
        async run(inputTensor) {
            try {
                // create a new session and load the model.
                const session = await ort.InferenceSession.create('/model/mnist-8.onnx',
                    { executionProviders: ['webgl'] })

                // prepare feeds. use model input names as keys.
                const feeds = { Input3: inputTensor }

                // feed inputs and run
                const results = await session.run(feeds)
                let data = results.Plus214_Output_0.data
                let digitResult = this.softMax(data)
                    .reduce((iMax, x, i, arr) => x > arr[iMax] ? i : iMax, 0)
                this.updateVolume(digitResult * 11 / 100)
            } catch (e) {
                console.log(e);
            }
        },

        // Post processing model output
        softMax(arr) {
            const C = Math.max(...arr)
            const d = arr.map((y) => Math.exp(y - C)).reduce((a, b) => a + b)
            return arr.map((value) => {
                return Math.exp(value - C) / d;
            });
        },

        onMouseUp() {
            this.onSelectEnd()
            this.onDotMoveEnd()
        },

        onDotMoveStart(e) {
            if (!this.moving) {
                this.moving = true
                this.movingDotId = e.target.id
                // console.log("Dot move start: ", this.movingDotId)
                this.dotStart.x = e.clientX
                this.dotStart.y = e.clientY
                this.dotPosition.left = parseInt(getComputedStyle(e.target).left)
                this.dotPosition.top = parseInt(getComputedStyle(e.target).top)
            }
        },

        onDotMoveEnd() {
            if (this.moving) {
                // console.log("Dot move end")
                this.moving = false
                this.movingDotId = ""
                this.dotStart = { x: 0, y: 0 }
                this.dotPosition = { left: 0, top: 0 }
            }
        },

        onDotMoving(e) {
            if (this.moving) {
                let currentLeft = this.dotPosition.left + e.clientX - this.dotStart.x
                let currentTop = this.dotPosition.top + e.clientY - this.dotStart.y
                if (currentLeft < 0 || currentLeft > (CANVAS_W - DOT_RADIUS * 2)
                    || currentTop < 0 || currentTop > (CANVAS_W - DOT_RADIUS * 2)) {
                    this.onDotMoveEnd()
                    return
                }
                let dot = document.getElementById(this.movingDotId)
                if (dot) {
                    dot.style.top = `${currentTop}px`
                    dot.style.left = `${currentLeft}px`
                }
            }
        },

        onSelectStart(e) {
            if (!this.selecting) {
                // console.log("Select start")
                this.selecting = true
                this.selectStart.x = e.clientX
                this.selectStart.y = e.clientY
            }
        },

        onSelectEnd() {
            if (this.selecting) {
                // console.log("Select end")
                this.selecting = false
            }
        },

        onSelectMoving(e) {
            if (this.selecting) {
                let area = document.getElementById("pattern-selector")
                area.hidden = 0
                let currX = e.clientX, currY = e.clientY
                let currLeft = Math.min(currX, this.selectStart.x)
                let currTop = Math.min(currY, this.selectStart.y)
                let currW = Math.abs(currX - this.selectStart.x)
                let currH = Math.abs(currY - this.selectStart.y)
                area.style.left = `${currLeft}px`
                area.style.top = `${currTop}px`
                area.style.width = `${currW}px`
                area.style.height = `${currH}px`
            }
        }
    }
}
</script>

<template>
<div class="control-container">
    <div>Volume Level (0-9): {{ Math.floor(this.volume * 100 / 11) }}</div>
    <div id="pattern-container"
        @mouseup="onMouseUp"
        @mousedown.shift="onSelectStart($event)"
        @mousemove.shift="onSelectMoving($event)"
        @mousemove.exact="onDotMoving($event)"
        @mouseleave="onDotMoveEnd">
        <div
            v-for="i in 100"
            :key="i"
            :id="`dot-${i}`"
            class="dot"
            @mousedown="onDotMoveStart($event)"
            :style="`top: ${this.dots[i-1][1]}px;
                left: ${this.dots[i-1][0]}px`">
        </div>
    </div>
    <div id="pattern-selector" hidden></div>
    <!-- <div>
        <img id="scaled-img">
    </div> -->
    <button @click="recognize">Recognize volume</button>
</div>
</template>

<style scoped>
.control-container {
    text-align: center;
}
#pattern-container {
    border: 1px solid black;
    margin: auto;
    width: 500px;
    height: 500px;
    position: relative;
}
.dot {
    position: absolute;
    background: black;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    cursor: pointer;
}
#pattern-selector {
    position: absolute;
    top: 0;
    left: 0;
    border: 1px solid black;
    pointer-events: none;
}
</style>