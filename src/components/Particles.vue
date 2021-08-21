<template>
  <canvas ref="canvasRef" class="webgl"></canvas>
</template>

<script lang="ts">
import { ref, defineComponent, onMounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import * as dat from "dat.gui"
import { BufferGeometry, Points, PointsMaterial } from 'three'

const gui = new dat.GUI()

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

      camera.position.set(20, 10, 6)
      scene.add(camera)

      const preferences = {
        particleQuantity: 20000,
        branches: 8,
        randomnessOnRadiusExpanding: 1/10,
        randomnessAmplification: 2,
        radius: 20,
        particlesRotation: 0.7,
        innerColor: '#ff3c30',
        outerColor: '#1498bb'
      }

      let geometry: BufferGeometry = null
      let material: PointsMaterial = null
      let particles: Points = null
      let hasExistedGalaxy = false

      function generateGalaxy() {

        if (hasExistedGalaxy) {

          geometry.dispose()
          material.dispose()
          scene.remove(particles)

        }

        geometry = new THREE.BufferGeometry()
        const positions = new Float32Array(preferences.particleQuantity * 3)
        const colors = new Float32Array(preferences.particleQuantity * 3)
        const innerColor = new THREE.Color(preferences.innerColor)
        const outerColor = new THREE.Color(preferences.outerColor)

        function spinPoints([x, y], radians) {
          return [x * Math.sin(radians + x * preferences.particlesRotation), y * Math.cos(radians + y * preferences.particlesRotation)]
        }

        function random() {
          return Math.random() - 0.5
        }

        function generateRandomness(enlarge) {
          return Math.pow(random(), 3) + random() * enlarge
        }

        for (let i = 0; i < preferences.particleQuantity; i++) {

          const points = i * 3
          const spinFraction = i % preferences.branches
          const spinRadians = 2 * Math.PI * (spinFraction / preferences.branches)
          const originalPointRadius = Math.pow(Math.random(), 1.5) * preferences.radius
          const randomnessParameter = preferences.randomnessAmplification * Math.pow(originalPointRadius * preferences.randomnessOnRadiusExpanding, 2)
          const spinedPoints = spinPoints([originalPointRadius, originalPointRadius], spinRadians)
          const color = innerColor.clone().lerp(outerColor, Math.pow(originalPointRadius / preferences.radius, 1/1.5))

          positions[points] = spinedPoints[0] + generateRandomness(randomnessParameter)
          positions[points + 1] = Math.pow(generateRandomness(randomnessParameter), 3) + random() * (1 / (originalPointRadius * 1))
          positions[points + 2] = spinedPoints[1] + generateRandomness(randomnessParameter)
          colors[points] = color.r
          colors[points + 1] = color.g
          colors[points + 2] = color.b

        }

        geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
        geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))

        material = new THREE.PointsMaterial({
          size: 0.08,
          sizeAttenuation: true,
          vertexColors: true,
          depthWrite: false,
          blending: THREE.AdditiveBlending
        })

        particles = new THREE.Points(geometry, material)

        scene.add(particles)

        hasExistedGalaxy = true

      }

      generateGalaxy()

      gui.add(preferences, 'particleQuantity').min(5000).max(50000).step(1000).onFinishChange(generateGalaxy)
      gui.add(preferences, 'branches').min(2).max(20).step(1).onFinishChange(generateGalaxy)
      gui.add(preferences, 'randomnessOnRadiusExpanding').min(1/50).max(0.15).onFinishChange(generateGalaxy)
      gui.add(preferences, 'randomnessAmplification').min(1).max(10).step(1).onFinishChange(generateGalaxy)
      gui.add(preferences, 'radius').min(5).max(50).step(5).onFinishChange(generateGalaxy)
      gui.add(preferences, 'particlesRotation').min(0).max(4).onFinishChange(generateGalaxy)
      gui.addColor(preferences, 'innerColor').onFinishChange(generateGalaxy)
      gui.addColor(preferences, 'outerColor').onFinishChange(generateGalaxy)

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
