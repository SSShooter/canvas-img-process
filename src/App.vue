<template>
  <div id="app">
    <div class="system">
      <aside>
        <input id="file" type="file" @change="getCustomImage">
        <label for="file">选择文件</label>
        <div v-html="'&#128522;&#128521;&#128519;&#128518;&#128525;&#129315;&#128514;'"></div>
        <button @click="bOrA = !bOrA">{{bOrA?'before':'after'}}</button>
        <button @click="generateAsciiArt(69)">彩蛋code69</button>
        <button @click="generateAsciiArt(10)">彩蛋code10</button>
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
        <div class="divisor">
          divisor<input type="number" v-model="divisor">
        </div>
        <div>
          <button @click="setKernelAndDivisor([0,0,0,0,1,0,0,0,0])">重置</button>
          <button @click="setKernelAndDivisor([1,1,1,1,1,1,1,1,1],9)">快速模糊</button>
          <button @click="setKernelAndDivisor([1,2,1,2,4,2,1,2,1],16)">高斯模糊</button>
          <button @click="setKernelAndDivisor([0, -1, 0, -1, 5, -1, 0, -1, 0])">锐化</button><br>
          <button @click="setKernelAndDivisor([-2, -1, 0, -1, 1, 1, 0, 1, 2])">浮雕</button>
          <button @click="setKernelAndDivisor([0, 0, 0, -1, 1, 0, 0, 0, 0])">边缘增强</button>
          <button @click="setKernelAndDivisor( [0, 1, 0, 1, -4, 1, 0, 1, 0])">边缘检测1</button>
          <button @click="setKernelAndDivisor( [-1, -1, -1, -1, 8, -1,-1, -1, -1])">边缘检测2</button>
        </div>
      </aside>
      <div class="canvas-display">
        <Spin v-if="loading" />
        <canvas id="input">
        </canvas>
        <canvas v-show="bOrA" id="output">
        </canvas>
      </div>
    </div>
    <!-- <div id="ascii-art"><div class="art" v-for="line in asciiArray"><span v-for="bit in line">{{bit}}</span></div></div> -->
  </div>
</template>

<script>
// TODO 添加反色，添加灰度
var FileSaver = require('file-saver')
import Spin from './Spin.vue'
export default {
  name: 'app',
  data() {
    return {
      loading: false,
      inputCtx: '',
      outputCtx: '',
      width: 800,
      height: 480,
      bOrA: true,
      asciiArray: [],
      divisor: 1,
      custom: [0, 0, 0, 0, 1, 0, 0, 0, 0],
      imageData: '' // 原图的imageData
    }
  },
  components: { Spin },
  watch: {
    custom(matrix) {
      matrix.map(val => Number(val) || 0)
      this.convolutionMatrix(this.imageData, this.custom, 0)
    },
    divisor(divisor) {
      this.convolutionMatrix(this.imageData, this.custom, 0)
    }
  },
  methods: {
    setKernelAndDivisor(kernel, divisor) {
      divisor = divisor || 1
      this.custom = kernel
      this.divisor = divisor
    },
    html2Canvas() {},
    getCustomImage(e) {
      let vm = this
      let file = e.target.files[0]
      let reader = new FileReader()
      reader.readAsDataURL(file)
      reader.addEventListener('load', () => {
        let img = new Image()
        img.src = reader.result
        img.onload = function() {
          vm.draw(this, this.width, this.height)
        }
      })
    },
    convolutionMatrix(input, m, offset=0) {
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
                this.divisor
          }
          oD[(y * w + x) * 4 + 3] = 255
        }
      }
      ctx.putImageData(output, 0, 0)
    },
    generateAsciiArt(grayScaleSelect) {
      let vm = this
      let canvas = document.getElementById('output')
      let ctx = canvas.getContext('2d')
      let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      console.log('get ctx')
      let data = imageData.data
      let width = canvas.width
      let widthSign = 1
      let heightSign = 1

      let grayScale2Ascii
      let grayScale

      if (grayScaleSelect === 69) {
        // 69级灰度
        grayScale2Ascii =
          '$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~<>i!lI;:,"^`\'. '
        grayScale = grayScale2Ascii.length
      } else {
        // 十级灰度
        grayScale2Ascii = '@%#*+=-:. '
        grayScale = grayScale2Ascii.length
      }
      console.log('get grayScale')
      let newCanvas = document.createElement('canvas')
      newCanvas.height = imageData.height * 10
      newCanvas.width = imageData.width * 10
      let newCtx = newCanvas.getContext('2d')
      newCtx.fillStyle = '#fff'
      newCtx.fillRect(0, 0, imageData.width * 10, imageData.height * 10)
      newCtx.fillStyle = '#000'
      newCtx.font = '15px'

      vm.asciiArray = []
      let asciiLineArray = []
      console.log('parse start')
      for (let i = 0; i < data.length; i += 4) {
        let avg = (data[i] + data[i + 1] + data[i + 2]) / 3
        asciiLineArray.push(
          grayScale2Ascii[Math.floor(avg / 225 * grayScale)] || ' '
        )
        newCtx.fillText(
          grayScale2Ascii[Math.floor(avg / 225 * grayScale)] || ' ',
          widthSign * 10,
          heightSign * 10
        )
        if (widthSign < width) {
          widthSign += 1
        } else {
          vm.asciiArray.push(asciiLineArray)
          asciiLineArray = []
          widthSign = 1
          heightSign += 1
        }
      }
      console.log('parse end')
      // vm.html2Canvas()
      let blob = newCanvas.toBlob(
        blob => FileSaver.saveAs(blob, 'hello world.jpg'),
        'image/jpeg',
        0.8
      )
    },
    draw(img, width, height) {
      this.loading = true
      let display = document.querySelector('.canvas-display')
      let input = document.querySelector('#input')
      let output = document.querySelector('#output')

      display.removeChild(input)
      display.removeChild(output)

      this.inputCtx = input.getContext('2d')

      this.outputCtx = output.getContext('2d')

      if (80 / 48 < width / height) {
        output.width = input.width = 800
        output.height = input.height = 800 * height / width
        let top = 240 - 400 * height / width
        output.style.top = input.style.top = top + 'px'
        display.appendChild(input)
        display.appendChild(output)
      } else {
        output.height = input.height = 480
        output.width = input.width = 480 * width / height
        let left = 400 - 240 * width / height
        output.style.left = input.style.left = left + 'px'
        display.appendChild(input)
        display.appendChild(output)
      }
      let ctx = this.inputCtx
      ctx.drawImage(img, 0, 0, input.width, input.height)
      let imageData = ctx.getImageData(0, 0, input.width, input.height)
      this.imageData = imageData
      this.convolutionMatrix(imageData, this.custom, 0)
      this.loading = false
    }
  },
  mounted() {
    let vm = this
    let img = new Image()
    img.src = './img.jpg'
    img.onload = function() {
      vm.draw(this, this.width, this.height)
    }
  }
}
</script>

<style lang="less">
@import './common.css';
#app {
  width: 1000px;
  margin: 50px auto;
  .system {
    display: flex;
    aside {
      width: 200px;
    }
    .canvas-display {
      position: relative;
      width: 800px;
      height: 480px;
      canvas {
        position: absolute;
      }
    }
    .matrix {
      margin: 5px;
      display: flex;
      flex-wrap: wrap;
      width: 120px;
      > input {
        box-sizing: border-box;
        height: 40px;
        width: 40px;
        text-align: center;
      }
    }
    .divisor {
      margin: 5px;
      > input {
        margin: 5px;
        height: 20px;
        width: 40px;
      }
    }
  }

  #ascii-art {
    display: inline-block;
    white-space: pre;
  }
  .art {
    font-size: 12px;
    line-height: 7px;
    font-family: Courier;
  }
}
</style>
