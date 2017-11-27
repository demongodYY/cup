<template>
  <div id="container">
    <el-dialog title="图片选择" width="540" :visible.sync="dialogVisible">
      <div class="thumbs">
        <el-row>
          <el-col :span="5" v-for="(item, index) in images" :key="index">
              <div class="img-btn">
                <el-button type="success" size="mini" round @click="drawImage(item);">选择图片</el-button>
                <br/>
                <el-button type="danger" size="mini" round @click="images.splice(index, 1)">删除图片</el-button>
              </div>
              <img :src="item" :alt="`保存的图片${index}`" class="savedImg">
          </el-col>
      </el-row>
      </div>
      <div slot="footer" class="dialog-footer">
        <input type="file" id="fileup" @change="uploadImage"/>
        <el-button @click="dialogVisible = false">取 消</el-button>
      </div>
    </el-dialog>
    <el-popover v-show="textShow" placement="right" width="210">
      <textarea
        slot="reference"
        ref="textarea"
        autofocus
        v-show="textShow">
      </textarea>
      <div>
          <el-select v-model="fontStyle.fontFamily" @change="changeTextStyle">
            <el-option
              v-for="item in fontFamilys"
              :key="item"
              :label="item"
              :value="item"
            >
            </el-option>
          </el-select>
          <el-color-picker v-model="fontStyle.color" @change="changeTextStyle"></el-color-picker>
          <el-input-number v-model="fontStyle.fontSize" label="字体" size="small" @change="changeTextStyle"></el-input-number> 
          <el-radio-group v-model="fontStyle.textAlign" size="medium" @change="changeTextStyle">
            <el-radio-button label="left">靠左</el-radio-button>
            <el-radio-button label="center">居中</el-radio-button>
            <el-radio-button label="right">靠右</el-radio-button>
          </el-radio-group>
      </div>
    </el-popover>
    <div id="myCanvas">
    </div>
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
      layer: '',
      stage: '',
      dialogVisible: false,
      currentGroup: '',
      fontStyle: {
        fontFamily: 'serif',
        fontSize: 30,
        textAlign: 'left',
        color: '#808080'
      },
      fontFamilys: [
        'serif',
        'sans-serif',
        'monospace',
        'cursive',
        'fantasy',
        'system-ui'
      ],
      images: []
    }
  },
  methods: {
    saveImage () {
      const dataUrl = this.stage.toDataURL()
      this.$root.bus.$emit('canvasChange', dataUrl)
    },
    changeTextStyle () {
      this.fontStyle.fontSize = `${this.fontStyle.fontSize}px`
      const textarea = this.$refs.textarea
      Object.assign(textarea.style, this.fontStyle)
    },
    printText (text) {
      const textarea = this.$refs.textarea
      const strSize = textarea.style.fontSize
      const numSize = Number(strSize.substring(0, strSize.length - 2))
      text.fontSize(numSize)
      text.fontFamily(textarea.style.fontFamily)
      text.text(textarea.value)
      text.align(textarea.style.textAlign)
      text.stroke(textarea.style.color)
      this.layer.draw()
      this.textShow = false
      this.saveImage()
    },
    inputText (text) {
      const textarea = this.$refs.textarea
      const textPosition = text.position()
      textarea.value = text.text()
      textarea.style.position = 'absolute'
      textarea.style.fontSize = `${text.fontSize()}px`
      textarea.style.fontFamily = text.fontFamily()
      textarea.style.color = text.stroke()
      textarea.style.textAlign = text.align()
      textarea.style.width = `${text.width()}px`
      textarea.style.height = `${text.height()}px`
      textarea.style.top = textPosition.y + 'px'
      textarea.style.left = textPosition.x + 'px'
      textarea.style['z-index'] = 1
      this.textShow = true
      textarea.addEventListener('keydown', (e) => {
        if (e.keyCode === 13) {
          this.printText(text)
        }
      })
    },
    drawImage (dataUrl) {
      let shape
      let sWidth
      let sHeight
      let pX
      let pY
      if (this.currentGroup.hasChildren('Circle')) {
        shape = this.currentGroup.findOne('Circle')
        sWidth = shape.size().width
        sHeight = shape.size().height
        pX = shape.position().x - sWidth / 2
        pY = shape.position().y - sHeight / 2
      } else if (this.currentGroup.hasChildren('Rect')) {
        shape = this.currentGroup.findOne('Rect')
        sWidth = shape.size().width
        sHeight = shape.size().height
        pX = shape.position().x
        pY = shape.position().y
      }
      const imgObj = new Image()
      imgObj.src = dataUrl
      imgObj.onload = () => {
        const img = new Konva.Image({
          name: 'img',
          image: imgObj,
          width: sWidth,
          height: sHeight,
          x: pX,
          y: pY,
          draggable: true
        })
        this.currentGroup.add(img)
        this.layer.draw()
        this.saveImage()
        this.dialogVisible = false
      }
    },
    uploadImage () {
      const fileUp = document.getElementById('fileup')
      const fr = new FileReader()
      fr.onload = (e) => {
        this.images.push(e.target.result)
      }
      fr.readAsDataURL(fileUp.files[0])
    },
    addImgShapes (imgObj) {
      if (imgObj.type === 'circle') {
        const group = new Konva.Group({
          clipFunc: (ctx) => {
            ctx.arc(imgObj.x, imgObj.y, imgObj.r, 0, Math.PI * 2)
          }
        })
        const circle = new Konva.Circle({
          x: imgObj.x,
          y: imgObj.y,
          radius: imgObj.r,
          stroke: 'black',
          strokeWidth: 2
        })
        group.add(circle)
        group.on('click', () => {
          this.currentGroup = group
          this.dialogVisible = true
        })
        this.layer.add(group)
        this.layer.draw()
      }
    },
    addTextShapes (textObj) {
      const text = new Konva.Text({
        x: textObj.x,
        y: textObj.y,
        width: textObj.width,
        height: textObj.height,
        stroke: '#808080',
        strokeWidth: 1,
        text: '点我输入文字',
        fontSize: 30,
        fontFamily: 'serif'
      })
      text.on('click', () => {
        this.inputText(text)
      })
      this.layer.add(text)
      this.layer.draw()
    }
  },
  mounted () {
    this.stage = new Konva.Stage({
      container: 'myCanvas',
      width: 800,
      height: 600
    })
    const bgLayer = new Konva.Layer()
    this.layer = new Konva.Layer()
    this.stage.add(bgLayer)
    this.stage.add(this.layer)
    bgLayer.moveToBottom()
    this.$root.bus.$on('bgChange', (bgUrl) => {
      bgLayer.find('.background').destroy()
      const bgImgObj = new Image()
      bgImgObj.src = bgUrl
      bgImgObj.onload = () => {
        const img = new Konva.Image({
          name: 'background',
          image: bgImgObj
        })
        bgLayer.add(img)
        img.moveToBottom()
        bgLayer.draw()
        this.saveImage()
      }
    })
    this.$root.bus.$on('layoutChange', (layout) => {
      this.layer.removeChildren()
      layout.img.map((item) => {
        this.addImgShapes(item)
      })
      layout.text.map((item) => {
        this.addTextShapes(item)
      })
      this.saveImage()
    })
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
  img.savedImg {
    position: absolute;
    max-width:100%;
    max-height:100%;
    left: 0;
    top: 0;
    bottom: 0;
    right: 0;
    margin: auto;
  }
  div.img-btn {
    opacity: 0;
    position: absolute;
    height: 50%;
    z-index: 2;
    left: 0;
    top: 0;
    bottom: 0;
    right: 0;
    margin: auto;
  }
  .el-button{
    margin:5px;
  }
  .el-col {
    position: relative;
    height: 166px;
    margin: 0 1.5%;
  }
  .el-col:hover {
    border: 8px solid #00B6C9
  }
  .el-col:hover .savedImg {
    opacity: 0.5;
  }
  .el-col:hover .img-btn {
    opacity: 1;
  }
</style>
