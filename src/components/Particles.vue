<template>
  <canvas ref="canvasRef" class="webgl"></canvas>
</template>

<script lang="ts">
import { ref, defineComponent, onMounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

export default defineComponent({
  name: 'Particles',
  setup: () => {

    const canvasRef = ref(null)

    onMounted(() => {

      const canvas = canvasRef.value
      const sizes = {
        width: window.innerWidth,
        height: window.innerHeight
      }
      const scene = new THREE.Scene()
      const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)

      camera.position.set(1, 1, 1)
      scene.add(camera)

      const geometry = new THREE.BufferGeometry()
      const count = 500
      const positions = new Float32Array(count * 3)

      for (let i = 0; i < count * 3; i++) {
        positions[i] = (Math.random() - 0.5) * 10
      }

      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

      const points = new THREE.PointsMaterial({
        size: 0.02,
        sizeAttenuation: true
      })

      const particles = new THREE.Points(geometry, points)

      scene.add(particles)

      const controls = new OrbitControls(camera, canvas)
      controls.enableDamping = true

      const renderer = new THREE.WebGLRenderer({
        canvas
      })
      
      renderer.setSize(sizes.width, sizes.height)
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

      window.addEventListener('resize', () => {
          // Update sizes
          sizes.width = window.innerWidth
          sizes.height = window.innerHeight

          // Update camera
          camera.aspect = sizes.width / sizes.height
          camera.updateProjectionMatrix()

          // Update renderer
          renderer.setSize(sizes.width, sizes.height)
          renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
      })

      const clock = new THREE.Clock()

      const tick = () => {
          controls.update()
          renderer.render(scene, camera)
          window.requestAnimationFrame(tick)
      }

      tick()

    })

    return {
      canvasRef
    }
  }
})
</script>

<style>
  canvas {
    display: block;
  }
</style>
