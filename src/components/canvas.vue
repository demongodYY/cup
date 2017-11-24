<template>
  <div>
    <div id="myCanvas"></div>
    <el-button plain>上一步</el-button>
    <el-button plain>下一步</el-button>
    <el-button plain>清空重做</el-button>
  </div>
</template>

<script>
import Konva from 'konva'

export default {
  name: 'cupCanvas',
  props: ['bgUrl'],
  mounted () {
    const stage = new Konva.Stage({
      container: 'myCanvas',
      width: 800,
      height: 600
    })
    const bgLayer = new Konva.Layer()
    // Konva.Image.fromURL(this.bgUrl, (image) => {
    //   bgLayer.add(image)
    //   stage.add(bgLayer)
    // })

    this.$root.bus.$on('bgChange', (text) => {
      bgLayer.remove()
      const bgImgObj = new Image()
      bgImgObj.src = text
      bgImgObj.onload = function () {
        const img = new Konva.Image({
          image: bgImgObj
        })
        bgLayer.add(img)
        stage.add(bgLayer)
      }
    })
    // const circle = new Konva.Circle({
    //   x: stage.getWidth() / 2,
    //   y: stage.getHeight() / 2,
    //   radius: 70,
    //   fill: 'red',
    //   stroke: 'black',
    //   strokeWidth: 4
    // })
    // layer.add(circle)
  }
}
</script>
<style scoped>
</style>
