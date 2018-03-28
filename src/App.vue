<template>
  <div id="app">
    <aside>
      <input type="file" @change="getCustomImage"><br>
      <button @click="bOrA = !bOrA">{{bOrA?'before':'after'}}</button>
      <button @click="generateAsciiArt">ascii art</button>
      <div class="matrix">
        <input type="number" v-model="custom[0]">
        <input type="number" v-model="custom[1]">
        <input type="number" v-model="custom[2]">
        <input type="number" v-model="custom[3]">
        <input type="number" v-model="custom[4]">
        <input type="number" v-model="custom[5]">
        <input type="number" v-model="custom[6]">
        <input type="number" v-model="custom[7]">
        <input type="number" v-model="custom[8]">
      </div>
      <div>
        <button @click="custom = [0,0,0,0,1,0,0,0,0]">重置</button>
        <button @click="custom = [0.0625, 0.125, 0.0625, 0.125, 0.25, 0.125, 0.0625, 0.125, 0.0625]">模糊</button>
        <button @click="custom = [0, -1, 0, -1, 5, -1, 0, -1, 0]">锐化</button>
        <button @click="custom = [-2, -1, 0, -1, 1, 1, 0, 1, 2]">浮雕</button>
        <button @click="custom = [0, 0, 0, -1, 1, 0, 0, 0, 0]">边缘增强</button>
        <button @click="custom =  [0, 1, 0, 1, -4, 1, 0, 0, 0]">边缘检测</button>
      </div>
    </aside>
    <div class="canvas-display">
      <canvas id="input" :width="width" :height="height">
      </canvas>
      <canvas v-show="bOrA" id="output" :width="width" :height="height">
      </canvas>
    </div>
    <canvas id="mini" :width="width/4" :height="height/4">
    </canvas>
    <pre><div class="art" v-for="line in asciiArray"><span v-for="bit in line">{{bit}}</span></div></pre>
  </div>
</template>

<script>
export default {
  name: 'app',
  data() {
    return {
      inputCtx: '',
      outputCtx: '',
      width: 800,
      height: 480,
      bOrA: true,
      asciiArray: [],
      custom: [0, 0, 0, 0, 1, 0, 0, 0, 0]
    }
  },
  watch: {
    custom(matrix) {
      matrix.map(val => Number(val) || 0)
      this.convolutionMatrix(this.imageData, this.custom, 1, 0)
    }
  },
  methods: {
    getCustomImage(e) {
      let vm = this
      let file = e.target.files[0]
      let reader = new FileReader()
      reader.readAsDataURL(file)
      reader.addEventListener('load', () => {
        let img = new Image()
        img.src = reader.result
        img.onload = function() {
          draw(this)
        }
        function draw(img) {
          let ctx = vm.inputCtx
          ctx.drawImage(img, 0, 0, vm.width, vm.height)
          let imageData = ctx.getImageData(0, 0, vm.width, vm.height)
          vm.imageData = imageData
          vm.convolutionMatrix(imageData, vm.custom, 1, 0)
        }
      })
    },
    convolutionMatrix(input, m, divisor, offset) {
      let ctx = this.outputCtx
      let output = ctx.createImageData(input)
      let w = input.width,
        h = input.height
      let iD = input.data,
        oD = output.data
      for (let y = 1; y < h - 1; y += 1) {
        for (let x = 1; x < w - 1; x += 1) {
          for (let c = 0; c < 3; c += 1) {
            let i = (y * w + x) * 4 + c
            oD[i] =
              offset +
              (m[0] * iD[i - w * 4 - 4] +
                m[1] * iD[i - w * 4] +
                m[2] * iD[i - w * 4 + 4] +
                m[3] * iD[i - 4] +
                m[4] * iD[i] +
                m[5] * iD[i + 4] +
                m[6] * iD[i + w * 4 - 4] +
                m[7] * iD[i + w * 4] +
                m[8] * iD[i + w * 4 + 4]) /
                divisor
          }
          oD[(y * w + x) * 4 + 3] = 255
        }
      }
      ctx.putImageData(output, 0, 0)
      document
        .getElementById('mini')
        .getContext('2d')
        .drawImage(
          document.getElementById('output'),
          0,
          0,
          this.width / 4,
          this.height / 4
        )
    },
    generateAsciiArt() {
      let vm = this
      let canvas = document.getElementById('mini')
      let ctx = canvas.getContext('2d')
      let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      let data = imageData.data
      let width = canvas.width
      let widthSign = 1
      let grayScale2Ascii =
        '$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~<>i!lI;:,"^`\'. '
      let grayScale = grayScale2Ascii.length
      vm.asciiArray = []
      let asciiLineArray = []
      for (let i = 0; i < data.length; i += 4) {
        let avg = (data[i] + data[i + 1] + data[i + 2]) / 3
        if (widthSign < width) {
          asciiLineArray.push(
            grayScale2Ascii[Math.floor(avg / 225 * grayScale)] || ' '
          )
          widthSign += 1
        } else {
          vm.asciiArray.push(asciiLineArray)
          asciiLineArray = []
          widthSign = 1
        }
      }
    }
  },
  mounted() {
    let vm = this
    vm.inputCtx = document.getElementById('input').getContext('2d')
    vm.outputCtx = document.getElementById('output').getContext('2d')
    let img = new Image()
    img.src = './img.jpg'
    img.onload = function() {
      draw(this)
    }

    function draw(img) {
      let ctx = vm.inputCtx
      ctx.drawImage(img, 0, 0, vm.width, vm.height)
      let imageData = ctx.getImageData(0, 0, vm.width, vm.height)
      vm.imageData = imageData
      vm.convolutionMatrix(imageData, vm.custom, 1, 0)
    }
  }
}
</script>

<style lang="less">
@import './common.css';
#app {
  display: flex;
  .canvas-display {
    position: relative;
    width: 800px;
    height: 480px;
    canvas {
      position: absolute;
    }
  }
  #mini {
    position: absolute;
    top: -1000px;
  }
  .art {
    font-size: 1px;
    line-height: 3px;
    font-family: Courier;
  }
  .matrix {
    display: flex;
    flex-wrap: wrap;
    width: 180px;
  }
  .matrix > input {
    box-sizing: border-box;
    height: 60px;
    width: 60px;
    text-align: center;
  }
}
</style>
