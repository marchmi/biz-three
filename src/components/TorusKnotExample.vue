<template>
  <div class="example-container">
    <div ref="canvasContainer" class="canvas-container"></div>
    <div class="info-panel">
      <h3>环形结 (TorusKnot)</h3>
      <p>复杂的几何体 - 环形结</p>
      <p>展示金属质感和镜面反射</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const canvasContainer = ref(null)
let scene, camera, renderer, torusKnot, animationId, lights

const initScene = () => {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x1a1a2e)

  camera = new THREE.PerspectiveCamera(
    75,
    canvasContainer.value.clientWidth / canvasContainer.value.clientHeight,
    0.1,
    1000
  )
  camera.position.z = 4

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(canvasContainer.value.clientWidth, canvasContainer.value.clientHeight)
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  renderer.toneMappingExposure = 1.5
  canvasContainer.value.appendChild(renderer.domElement)

  lights = [
    { light: new THREE.AmbientLight(0x404040, 0.3), position: [0, 0, 0] },
    { light: new THREE.DirectionalLight(0xffffff, 2), position: [5, 5, 5] },
    { light: new THREE.DirectionalLight(0x4ecdc4, 1), position: [-5, -5, -5] },
    { light: new THREE.PointLight(0xff6b6b, 3, 10), position: [0, 3, 0] }
  ]
  lights.forEach(({ light, position }) => {
    light.position.set(...position)
    scene.add(light)
  })

  const geometry = new THREE.TorusKnotGeometry(1, 0.3, 128, 32)
  const material = new THREE.MeshStandardMaterial({
    color: 0x667eea,
    roughness: 0.2,
    metalness: 0.8,
    envMapIntensity: 1
  })
  torusKnot = new THREE.Mesh(geometry, material)
  torusKnot.castShadow = true
  torusKnot.receiveShadow = true
  scene.add(torusKnot)

  const cubeTextureLoader = new THREE.CubeTextureLoader()
  const envMapColors = [
    '1a1a2e', '2d2d44', '667eea', '764ba2', '1a1a2e', '2d2d44'
  ]
  
  const canvas = document.createElement('canvas')
  canvas.width = 128
  canvas.height = 128
  const ctx = canvas.getContext('2d')
  const gradient = ctx.createLinearGradient(0, 0, 128, 128)
  gradient.addColorStop(0, '#667eea')
  gradient.addColorStop(1, '#764ba2')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, 128, 128)
  
  const texture = new THREE.CanvasTexture(canvas)
  texture.mapping = THREE.EquirectangularReflectionMapping
  scene.environment = texture
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  torusKnot.rotation.x += 0.01
  torusKnot.rotation.y += 0.015

  const time = Date.now() * 0.001
  torusKnot.scale.setScalar(1 + Math.sin(time * 2) * 0.05)

  const hue = (time * 20) % 360
  torusKnot.material.color.setHSL(hue / 360, 0.7, 0.5)

  camera.position.x = Math.sin(time * 0.5) * 0.5
  camera.position.y = Math.cos(time * 0.3) * 0.3
  camera.lookAt(0, 0, 0)

  renderer.render(scene, camera)
}

const handleResize = () => {
  if (!canvasContainer.value) return
  camera.aspect = canvasContainer.value.clientWidth / canvasContainer.value.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(canvasContainer.value.clientWidth, canvasContainer.value.clientHeight)
}

onMounted(() => {
  initScene()
  animate()
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  cancelAnimationFrame(animationId)
  window.removeEventListener('resize', handleResize)
  if (renderer) {
    renderer.dispose()
  }
})
</script>

<style scoped>
.example-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.canvas-container {
  flex: 1;
  width: 100%;
}

.info-panel {
  position: absolute;
  top: 20px;
  right: 20px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 15px 20px;
  border-radius: 10px;
  max-width: 250px;
}

.info-panel h3 {
  margin-bottom: 10px;
  color: #667eea;
}

.info-panel p {
  font-size: 0.85rem;
  margin: 5px 0;
  opacity: 0.9;
}
</style>
