<template>
  <div id="container">
      <textarea
        ref="textarea"
        autofocus
        v-show="textShow">
    </textarea>
    <div id="myCanvas">
    </div>
    <input type="file" id="fileup"/>
    <el-button plain disabled>上一步</el-button>
    <el-button plain disabled>下一步</el-button>
    <el-button plain disabled>清空重做</el-button>
  </div>
</template>

<script>
import Konva from 'konva'

export default {
  name: 'cupCanvas',
  data () {
    return {
      textShow: false,
      layer: ''
    }
  },
  methods: {
    saveImage (layer) {
      const dataUrl = layer.toDataURL()
      this.$root.bus.$emit('canvasChange', dataUrl)
    },
    inputText (text) {
      const textarea = this.$refs.textarea
      const textPosition = text.position()
      textarea.value = text.text()
      textarea.style.position = 'absolute'
      textarea.style.fontSize = `${text.fontSize()}px`
      textarea.style.fontFamily = text.fontFamily()
      textarea.style.width = `${text.width()}px`
      textarea.style.height = `${text.height()}px`
      textarea.style.top = textPosition.y + 'px'
      textarea.style.left = textPosition.x + 'px'
      textarea.style['z-index'] = 1
      this.textShow = true
      textarea.addEventListener('keydown', (e) => {
        if (e.keyCode === 13) {
          text.text(textarea.value)
          this.layer.draw()
          this.textShow = false
          this.saveImage(this.layer)
        }
      })
    },
    drawImage (dataUrl) {
      const imgObj = new Image()
      imgObj.src = dataUrl
      imgObj.onload = () => {
        const img = new Konva.Image({
          name: 'img',
          image: imgObj,
          width: 200,
          height: 200,
          x: this.layer.getWidth() / 2 - 100,
          y: 100,
          draggable: true
        })
        const group = new Konva.Group({
          clipFunc: (ctx) => {
            ctx.arc(this.layer.getWidth() / 2, 200, 100, 0, Math.PI * 2)
          }
        })
        group.add(img)
        this.layer.add(group)
        this.layer.draw()
        this.saveImage(this.layer)
      }
    },
    uploadImage () {
      const fileUp = document.getElementById('fileup')
      fileUp.onchange = () => {
        const fr = new FileReader()
        fr.onload = (e) => {
          this.drawImage(e.target.result)
        }
        fr.readAsDataURL(fileUp.files[0])
      }
    },
    addShapes () {
      const circle = new Konva.Circle({
        x: this.layer.getWidth() / 2,
        y: 200,
        radius: 100,
        stroke: 'blue',
        strokeWidth: 1
      })
      const text = new Konva.Text({
        x: (this.layer.getWidth() / 2) - 100,
        y: 400,
        width: 200,
        height: 100,
        stroke: 'blue',
        strokeWidth: 1,
        text: 'Hello World!',
        fontSize: 30,
        fontFamily: 'Calibri'
      })
      text.on('click', () => {
        this.inputText(text)
      })
      this.layer.add(circle)
      this.layer.add(text)
      this.layer.draw()
    }
  },
  mounted () {
    const stage = new Konva.Stage({
      container: 'myCanvas',
      width: 800,
      height: 600
    })
    const bgLayer = new Konva.Layer()
    this.layer = bgLayer
    stage.add(bgLayer)
    this.$root.bus.$on('bgChange', (text) => {
      bgLayer.find('.background').destroy()
      const bgImgObj = new Image()
      bgImgObj.src = text
      bgImgObj.onload = () => {
        const img = new Konva.Image({
          name: 'background',
          image: bgImgObj
        })
        bgLayer.add(img)
        img.moveToBottom()
        bgLayer.draw()
        this.saveImage(this.layer)
      }
    })
    this.addShapes()
    this.uploadImage()
  }
}
</script>
<style scoped>
  #container {
    position: relative;
  }
  #myCanvas {
    position: relative;
  }
</style>
