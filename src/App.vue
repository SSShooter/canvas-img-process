<template>
  <div id="app">
    <div class="system">
      <aside>
        <input type="file" @change="getCustomImage">
        <div v-html="'&#128522;&#128521;&#128519;&#128578;&#128518;&#128579;&#128525;&#129315;&#128514;'"></div>
        <button @click="bOrA = !bOrA">{{bOrA?'before':'after'}}</button>
        <button @click="generateAsciiArt">彩蛋</button>
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
        divisor<input type="number" v-model="divisor">
        <div>
          <button @click="setKernelAndDivisor([0,0,0,0,1,0,0,0,0])">重置</button>
          <button @click="setKernelAndDivisor([1,1,1,1,1,1,1,1,1],9)">模糊</button>
          <button @click="setKernelAndDivisor([0, -1, 0, -1, 5, -1, 0, -1, 0])">锐化</button>
          <button @click="setKernelAndDivisor([-2, -1, 0, -1, 1, 1, 0, 1, 2])">浮雕</button>
          <button @click="setKernelAndDivisor([0, 0, 0, -1, 1, 0, 0, 0, 0])">边缘增强</button>
          <button @click="setKernelAndDivisor( [0, 1, 0, 1, -4, 1, 0, 1, 0])">边缘检测</button>
        </div>
      </aside>
      <div class="canvas-display">
        <canvas id="input" :width="width" :height="height">
        </canvas>
        <canvas v-show="bOrA" id="output" :width="width" :height="height">
        </canvas>
      </div>
    </div>
    <!-- <div id="ascii-art">
      <div class="art" v-for="line in asciiArray">
        <span v-for="bit in line">{{bit}}</span>
      </div>
    </div> -->
  </div>
</template>

<script>
var FileSaver = require('file-saver')
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
      divisor: 1,
      custom: [0, 0, 0, 0, 1, 0, 0, 0, 0]
    }
  },
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
    html2Canvas() {
    },
    getCustomImage(e) {
      let vm = this
      let file = e.target.files[0]
      let reader = new FileReader()
      reader.readAsDataURL(file)
      reader.addEventListener('load', () => {
        let img = new Image()
        img.src = reader.result
        img.onload = function() {
          draw(this,this.width,this.height)
        }
        function draw(img) {
          let ctx = vm.inputCtx
          ctx.drawImage(img, 0, 0, vm.width, vm.height)
          let imageData = ctx.getImageData(0, 0, vm.width, vm.height)
          vm.imageData = imageData
          vm.convolutionMatrix(imageData, vm.custom, 0)
        }
      })
    },
    convolutionMatrix(input, m, offset) {
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
    generateAsciiArt() {
      // document
      //   .getElementById('mini')
      //   .getContext('2d')
      //   .drawImage(
      //     document.getElementById('output'),
      //     0,
      //     0,
      //     this.width / 4,
      //     this.height / 4
      //   )
      let vm = this
      let canvas = document.getElementById('output')
      console.log('get ctx')
      let ctx = canvas.getContext('2d')
      let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      let data = imageData.data
      let width = canvas.width
      let widthSign = 1
      let heightSign = 1

      // 69级灰度
      // let grayScale2Ascii =
      //   '$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~<>i!lI;:,"^`\'. '
      // let grayScale = grayScale2Ascii.length

      // 十级灰度
      let grayScale2Ascii = '@%#*+=-:. '
      let grayScale = grayScale2Ascii.length
      console.log('get grayScale')

      let newCanvas = document.createElement('canvas')
      newCanvas.height = imageData.height * 8
      newCanvas.width = imageData.width * 8
      let newCtx = newCanvas.getContext('2d')
      newCtx.fillStyle = '#fff'
      newCtx.fillRect(0, 0, imageData.width * 8, imageData.height * 8)
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
          widthSign * 8,
          heightSign * 8
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
.system {
  display: flex;
  .canvas-display {
    position: relative;
    width: 800px;
    height: 480px;
    canvas {
      position: absolute;
    }
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

#ascii-art {
  // position: absolute;
  // left: 100vw;
  display: inline-block;
  white-space: pre;
}
.art {
  font-size: 12px;
  line-height: 7px;
  font-family: Courier;
}
</style>
