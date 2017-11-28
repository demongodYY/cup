<template>
  <div id="container">
    <div id="p-container" class="container">
    </div>
    <div id="images-back">
      <img src="/static/img/cup-back.png" />
       <el-slider v-model="degree" :max="180" :min="-180"></el-slider>
    </div>
    <div id="images-front">
      <img src="/static/img/cup-front.png" />
    </div>
  </div>
</template>

<script>
  import * as THREE from 'three'
  export default {
    name: 'preview',
    data () {
      return {
        scene: null,
        camera: null,
        renderer: null,
        cylinderMesh: null,
        degree: 0
      }
    },
    methods: {
      initScene () {
        const container = document.getElementById('p-container')
        this.scene = new THREE.Scene()
        this.camera = new THREE.PerspectiveCamera(45, 225 / 475, 1, 2000)
        this.camera.position.y = 400
        this.scene.add(this.camera)
        this.renderer = new THREE.WebGLRenderer({alpha: true})
        this.renderer.setPixelRatio(window.devicePixelRatio)
        this.renderer.setSize(225, 475)
        container.appendChild(this.renderer.domElement)
        const ambientLight = new THREE.AmbientLight(0xffffff, 1)
        this.scene.add(ambientLight)
      },
      renderCup () {
        const cylinderGeo = new THREE.CylinderGeometry(155, 128, 585, 64, 5, true)
        const texture = new THREE.TextureLoader().load('/static/img/bg1.png', (tex) => {
          texture.anisotropy = 2
          const cylinderMat = new THREE.MeshPhongMaterial({
            map: texture,
            side: THREE.DoubleSide,
            opacity: 0.8,
            transparent: true
          })
          this.cylinderMesh = new THREE.Mesh(cylinderGeo, cylinderMat)
          this.cylinderMesh.position.set(0, -40, 0)
          this.scene.add(this.cylinderMesh)
          this.camera.position.z = 800
          this.camera.lookAt(this.scene.position)
          const animate = () => {
            requestAnimationFrame(animate)
            this.cylinderMesh.rotation.y = this.degree * Math.PI / 180
            this.renderer.render(this.scene, this.camera)
          }
          animate()
          this.$root.bus.$on('canvasChange', (url) => {
            this.updateTexture(url)
          })
        })
      },
      updateTexture (url) {
        this.cylinderMesh.material.map = new THREE.TextureLoader().load(url)
        this.cylinderMesh.material.needsUpdate = true
      }
    },
    mounted () {
      this.initScene()
      this.renderCup()
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  div#container {
    width:100%;
    height:100%;
    position: relative;
  }
  div#p-container {
    width: 225px;
    height: 475px;
    z-index: 2;
    position: absolute;
  }
  div#images-back{
    position: absolute;
    z-index: 1;
  }
  div#images-front{
    position: absolute;
    z-index: 3;
  }
  canvas { 
    width: 100%; 
    height: 100%; 
  }
</style>