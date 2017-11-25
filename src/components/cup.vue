<template>
  <div class="cup">
    <el-popover ref="bg-popup" placement= "right" trigger= "click" width="400" v-model="bgListVisible">
      <el-row :gutter="24">
        <el-col :span="12" v-for="(item, index) in bgList" :key="index">
          <el-button @click="changeBackground(index)">
            <img :src="item.url" :alt="`背景图${index}`">
          </el-button>
        </el-col>
      </el-row>
    </el-popover>
    <el-popover ref="layout-popup" placement= "right" trigger= "click" width="400">
      <el-row :gutter="24">
        <el-col :span="12" v-for="(item, index) in layoutList" :key="index">
          <el-button @click="changeBackground(index)">
            <img :src="item.url" :alt="`layout${index}`">
          </el-button>
        </el-col>
      </el-row>
    </el-popover>
    <el-container>
      <el-aside>
        <ul>
          <li>
            <el-button plain v-popover:bg-popup>更换背景</el-button>
          </li>
          <li>
            <el-button plain v-popover:layout-popup disabled="true">更换布局</el-button>
          </li>
        </ul>
      </el-aside>
      <el-main>
        <my-canvas></my-canvas>
      </el-main>
      <el-aside>
        <preview />
        <!-- <img src="/static/img/tervis2.png" alt="杯子"> -->
      </el-aside>
    </el-container>
  </div>
</template>

<script>
  import myCanvas from './canvas'
  import preview from './preview'
  const bl = require('../model/bg-list.json')
  const ll = require('../model/layout-list.json')
  export default {
    name: 'cup',
    components: {
      myCanvas,
      preview
    },
    methods: {
      changeBackground (index) {
        this.bgUrl = this.bgList[index].url
        this.bgListVisible = false
        this.$root.bus.$emit('bgChange', this.bgUrl)
      }
    },
    beforeMount () {
      this.bgList = bl
      this.layoutList = ll
    },
    mounted () {
      this.changeBackground(0)
    },
    data () {
      return {
        bgListVisible: false,
        bgList: [],
        bgUrl: '',
        layoutListVisible: false,
        layoutList: []
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  li {
    list-style-type: none;
  }
  img {
    width: 100%;
    height: auto;
  }
</style>
