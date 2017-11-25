<template>
  <div id="p-container" class="container">

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
        renderer: null
      }
    },
    methods: {
      initScene () {
        const container = document.getElementById('p-container')
        this.scene = new THREE.Scene()
        this.scene.background = new THREE.Color()
        this.camera = new THREE.PerspectiveCamera(75, container.clientWidth / window.innerHeight, 0.1, 1000)
        this.renderer = new THREE.WebGLRenderer()
        this.renderer.setSize(container.clientWidth, window.innerHeight)
        container.appendChild(this.renderer.domElement)
      },
      renderCup () {
        const cylinderGeo = new THREE.CylinderGeometry(1, 1, 5, 32)
        const texture = new THREE.TextureLoader().load('/static/img/bg1.png', (tex) => {
          const cylinderMat = new THREE.MeshBasicMaterial({
            map: texture
          })
          texture.needsUpdate = true
          const cylinderMesh = new THREE.Mesh(cylinderGeo, cylinderMat)
          this.scene.add(cylinderMesh)
          this.camera.position.z = 6
          const animate = () => {
            requestAnimationFrame(animate)
            cylinderMesh.rotation.y += 0.01
            this.renderer.render(this.scene, this.camera)
          }
          animate()
          this.$root.bus.$on('canvasChange', (url) => {
            this.updateTexture(cylinderMesh, url)
          })
          // setTimeout(() => {
          //   this.updateTexture(cylinderMesh)
          // }, 5000)
        })
      },
      updateTexture (mesh, url) {
        mesh.material.map = new THREE.TextureLoader().load(url)
        mesh.material.needsUpdate = true
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
  div#p-container {
    width: 100%;
    height: 100%;
  }
  canvas { 
    width: 100%; 
    height: 100%; 
  }
</style>