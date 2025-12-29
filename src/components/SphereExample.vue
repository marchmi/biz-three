<template>
  <div class="example-container">
    <div ref="canvasContainer" class="canvas-container"></div>
    <div class="info-panel">
      <h3>球体与光照</h3>
      <p>展示不同类型的光源效果</p>
      <p>包含：环境光、平行光、点光源</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const canvasContainer = ref(null)
let scene, camera, renderer, spheres, animationId, pointLight

const initScene = () => {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x0f0f23)

  camera = new THREE.PerspectiveCamera(
    75,
    canvasContainer.value.clientWidth / canvasContainer.value.clientHeight,
    0.1,
    1000
  )
  camera.position.z = 8

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(canvasContainer.value.clientWidth, canvasContainer.value.clientHeight)
  renderer.shadowMap.enabled = true
  canvasContainer.value.appendChild(renderer.domElement)

  const ambientLight = new THREE.AmbientLight(0x404040, 0.5)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 5, 5)
  directionalLight.castShadow = true
  scene.add(directionalLight)

  pointLight = new THREE.PointLight(0xff6b6b, 2, 20)
  pointLight.position.set(0, 0, 5)
  scene.add(pointLight)
  scene.add(new THREE.PointLightHelper(pointLight, 0.5))

  spheres = []
  const colors = [0xff6b6b, 0x4ecdc4, 0xffe66d, 0x95e1d3]
  const materials = [
    new THREE.MeshBasicMaterial({ color: 0xff6b6b }),
    new THREE.MeshLambertMaterial({ color: 0x4ecdc4 }),
    new THREE.MeshPhongMaterial({ color: 0xffe66d, shininess: 100 }),
    new THREE.MeshStandardMaterial({ color: 0x95e1d3, roughness: 0.3, metalness: 0.2 })
  ]

  const positions = [
    [-3, 0, 0],
    [-1, 0, 0],
    [1, 0, 0],
    [3, 0, 0]
  ]

  for (let i = 0; i < 4; i++) {
    const geometry = new THREE.SphereGeometry(0.7, 32, 32)
    const sphere = new THREE.Mesh(geometry, materials[i])
    sphere.position.set(...positions[i])
    sphere.castShadow = true
    sphere.receiveShadow = true
    spheres.push(sphere)
    scene.add(sphere)
  }

  const planeGeometry = new THREE.PlaneGeometry(20, 20)
  const planeMaterial = new THREE.MeshStandardMaterial({ 
    color: 0x2d2d44, 
    roughness: 0.8,
    metalness: 0.2
  })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2
  plane.position.y = -2
  plane.receiveShadow = true
  scene.add(plane)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  spheres.forEach((sphere, index) => {
    sphere.position.y = Math.sin(Date.now() * 0.002 + index) * 0.5
    sphere.rotation.y += 0.01
  })

  pointLight.position.x = Math.sin(Date.now() * 0.001) * 5
  pointLight.position.y = Math.cos(Date.now() * 0.0015) * 2

  camera.position.x = Math.sin(Date.now() * 0.0005) * 2
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
