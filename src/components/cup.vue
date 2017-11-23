<template>
  <div class="cup">
    <el-popover ref="bg-popup" placement= "right" trigger= "click" width="400">

    </el-popover>
    <el-popover ref="layout-popup" placement= "right" trigger= "click" width="400">

    </el-popover>
    <el-container>
      <el-side>
        <ul>
          <li>
            <el-button plain>更换背景</el-button>
          </li>
          <li>
            <el-button plain>更换布局</el-button>
          </li>
        </ul>
      </el-side>
      <el-main>
        <img src="/static/img/bg1.png" alt="背景图">
        <el-button plain>上一步</el-button>
        <el-button plain>下一步</el-button>
        <el-button plain>清空重做</el-button>
      </el-main>
      <el-side>
        <img src="/static/img/tervis2.png" alt="杯子">
      </el-side>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'cup',
  data () {
    return {
      bgUrl: '/static/img/bg1.png'
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  li {
    list-style-type: none;
  }
</style>
