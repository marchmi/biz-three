<template>
  <div class="example-container">
    <div ref="canvasContainer" class="canvas-container"></div>
    <div class="info-panel">
      <h3>旋转立方体</h3>
      <p>带有颜色渐变效果的立方体动画</p>
      <p>使用了MeshPhongMaterial材质</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const canvasContainer = ref(null)
let scene, camera, renderer, cube, animationId, lights

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
  renderer.setPixelRatio(window.devicePixelRatio)
  canvasContainer.value.appendChild(renderer.domElement)

  lights = [
    { light: new THREE.AmbientLight(0x404040, 0.5), position: [0, 0, 0] },
    { light: new THREE.DirectionalLight(0xffffff, 1), position: [5, 5, 5] },
    { light: new THREE.DirectionalLight(0xff6b6b, 0.5), position: [-5, -5, 5] },
    { light: new THREE.DirectionalLight(0x4ecdc4, 0.5), position: [5, -5, -5] }
  ]
  lights.forEach(({ light }) => scene.add(light.light))

  const geometry = new THREE.BoxGeometry(1.5, 1.5, 1.5)
  const material = new THREE.MeshPhongMaterial({
    color: 0x667eea,
    specular: 0x444444,
    shininess: 30
  })
  cube = new THREE.Mesh(geometry, material)
  scene.add(cube)

  const wireframe = new THREE.LineSegments(
    new THREE.EdgesGeometry(geometry),
    new THREE.LineBasicMaterial({ color: 0xffffff, opacity: 0.3, transparent: true })
  )
  cube.add(wireframe)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)
  
  cube.rotation.x += 0.01
  cube.rotation.y += 0.02
  
  cube.position.x = Math.sin(Date.now() * 0.001) * 0.5
  cube.position.y = Math.cos(Date.now() * 0.0015) * 0.3
  
  const hue = (Date.now() * 0.05) % 360
  cube.material.color.setHSL(hue / 360, 0.7, 0.5)
  
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
