<template>
  <div id="app">
    <input type="file" @change="getCustomImage"><br>
    <button @click="bOrA = !bOrA">before</button>
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
    <div class="canvas-display">
      <canvas id="input" :width="width" :height="height">
      </canvas>
      <canvas v-show="bOrA" id="output" :width="width" :height="height">
      </canvas>
    </div>
    <pre><div class="art" v-for="line in asciiArray"><span v-for="bit in line">{{bit}}</span></div></pre>

  </div>
</template>

<script>
export default {
  name: 'app',
  data() {
    return {
      width: 800,
      height: 480,
      bOrA:true,
      asciiArray: [],
      custom: [0, 0, 0, 0, 1, 0, 0, 0, 0]
    }
  },
  watch: {
    custom(matrix) {
      matrix.map(val => Number(val) || 0)
      this.ConvolutionMatrix(this.imageData, this.custom, 1, 0)
    }
  },
  methods: {
    getCustomImage(e) {
      let file = e.target.files[0]
      let reader = new FileReader()
      reader.readAsDataURL(file)
      reader.addEventListener('load', () => {
        let vm = this
        var img = new Image()
        img.src = reader.result
        img.onload = function() {
          draw(this)
        }

        function draw(img) {
          var canvas = document.getElementById('input')
          var ctx = canvas.getContext('2d')
          ctx.drawImage(img, 0, 0, vm.width, vm.height)
          var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
          vm.imageData = imageData
          vm.ConvolutionMatrix(imageData, vm.custom, 1, 0)
        }
      })
    },
    ConvolutionMatrix(input, m, divisor, offset) {
      var canvas = document.getElementById('output')
      var ctx = canvas.getContext('2d')
      var output = ctx.createImageData(input)
      var w = input.width,
        h = input.height
      var iD = input.data,
        oD = output.data
      // 对除了边缘的点之外的内部点的 RGB 进行操作，透明度在最后都设为 255
      for (var y = 1; y < h - 1; y += 1) {
        for (var x = 1; x < w - 1; x += 1) {
          for (var c = 0; c < 3; c += 1) {
            var i = (y * w + x) * 4 + c
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
          oD[(y * w + x) * 4 + 3] = 255 // 设置透明度为不透明
        }
      }
      ctx.putImageData(output, 0, 0)
    },
    generateAsciiArt() {
      let vm = this
      var canvas = document.getElementById('output')
      var ctx = canvas.getContext('2d')
      var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      var data = imageData.data
      let width = this.width
      let widthSign = 1
      let grayScale2Ascii =
        '$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~<>i!lI;:,"^`\'. '
      let grayScale = grayScale2Ascii.length
      vm.asciiArray = []
      let asciiLineArray = []
      for (var i = 0; i < data.length; i += 4) {
        var avg = (data[i] + data[i + 1] + data[i + 2]) / 3
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
    var img = new Image()
    img.src = './img.jpg'
    img.onload = function() {
      draw(this)
    }

    function draw(img) {
      var canvas = document.getElementById('input')
      var ctx = canvas.getContext('2d')
      ctx.drawImage(img, 0, 0, vm.width, vm.height)
      var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      vm.imageData = imageData
      vm.ConvolutionMatrix(imageData, vm.custom, 1, 0)
    }
  }
}
</script>

<style lang="less">
.canvas-display {
  position: relative;
  canvas {
    position: absolute;
  }
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
</style>
