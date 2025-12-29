<template>
  <div class="example-container">
    <div ref="canvasContainer" class="canvas-container"></div>
    <div class="info-panel">
      <h3>基础场景</h3>
      <p>这是最简单的Three.js场景，包含一个立方体</p>
      <p>场景元素：立方体 + 白色背景</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const canvasContainer = ref(null)
let scene, camera, renderer, cube, animationId

const initScene = () => {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0xffffff)

  camera = new THREE.PerspectiveCamera(
    75,
    canvasContainer.value.clientWidth / canvasContainer.value.clientHeight,
    0.1,
    1000
  )
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(canvasContainer.value.clientWidth, canvasContainer.value.clientHeight)
  canvasContainer.value.appendChild(renderer.domElement)

  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshBasicMaterial({ color: 0x667eea })
  cube = new THREE.Mesh(geometry, material)
  scene.add(cube)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)
  cube.rotation.x += 0.01
  cube.rotation.y += 0.01
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
