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

      camera.position.set(4, 3, 6)
      scene.add(camera)

      const geometry = new THREE.BufferGeometry()
      const count = 10000
      const branches = 6
      const positions = new Float32Array(count * 3)
      const randoemnesOnRadiusEnlarge = 3
      const randoemnesOnRadius = 1/6
      const radius = 10
      const pointsOffset = 1/2

      function spinPoints([x, y], radians) {
        return [x * Math.sin(radians + x * pointsOffset), y * Math.cos(radians + y * pointsOffset)]
      }

      function random() {
        return Math.random() - 0.5
      }

      function generateRandomness(enlarge) {
        return Math.pow(random(), 3) + random() * enlarge
      }

      for (let i = 0; i < count; i++) {

        const points = i * 3
        const spinFraction = i % branches
        const spinRadians = 2 * Math.PI * (spinFraction / branches)
        const originalPointRadius = Math.pow(Math.random(), 1.5) * radius
        const randomnessParameter = randoemnesOnRadiusEnlarge * Math.pow(originalPointRadius * randoemnesOnRadius, 2)
        const spinedPoints = spinPoints([originalPointRadius, originalPointRadius], spinRadians)

        positions[points] = spinedPoints[0] + generateRandomness(randomnessParameter)
        positions[points + 1] = Math.pow(generateRandomness(randomnessParameter), 3) + random() * (1 / (originalPointRadius * 4))
        positions[points + 2] = spinedPoints[1] + generateRandomness(randomnessParameter)

      }

      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

      const points = new THREE.PointsMaterial({
        size: 0.02,
        sizeAttenuation: true,
        color: new THREE.Color('white'),
        depthWrite: false,
        blending: THREE.AdditiveBlending
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
