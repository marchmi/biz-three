<template>
  <div class="example-container">
    <div ref="canvasContainer" class="canvas-container"></div>
    <div class="info-panel">
      <h3>粒子系统</h3>
      <p>使用BufferGeometry创建粒子</p>
      <p>粒子数量：{{ particleCount }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

const canvasContainer = ref(null)
const particleCount = 5000
let scene, camera, renderer, particles, animationId

const initScene = () => {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x0f0f23)
  scene.fog = new THREE.Fog(0x0f0f23, 5, 20)

  camera = new THREE.PerspectiveCamera(
    75,
    canvasContainer.value.clientWidth / canvasContainer.value.clientHeight,
    0.1,
    1000
  )
  camera.position.z = 8

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(canvasContainer.value.clientWidth, canvasContainer.value.clientHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  canvasContainer.value.appendChild(renderer.domElement)

  const geometry = new THREE.BufferGeometry()
  const positions = new Float32Array(particleCount * 3)
  const colors = new Float32Array(particleCount * 3)
  const sizes = new Float32Array(particleCount)

  const color1 = new THREE.Color(0x667eea)
  const color2 = new THREE.Color(0x764ba2)
  const color3 = new THREE.Color(0xff6b6b)

  for (let i = 0; i < particleCount; i++) {
    const i3 = i * 3

    const radius = 3 + Math.random() * 2
    const theta = Math.random() * Math.PI * 2
    const phi = Math.random() * Math.PI

    positions[i3] = radius * Math.sin(phi) * Math.cos(theta)
    positions[i3 + 1] = radius * Math.sin(phi) * Math.sin(theta)
    positions[i3 + 2] = radius * Math.cos(phi)

    const mixedColor = color1.clone()
    if (Math.random() > 0.5) {
      mixedColor.lerp(color2, Math.random())
    } else {
      mixedColor.lerp(color3, Math.random())
    }

    colors[i3] = mixedColor.r
    colors[i3 + 1] = mixedColor.g
    colors[i3 + 2] = mixedColor.b

    sizes[i] = Math.random() * 2 + 0.5
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
  geometry.setAttribute('size', new THREE.BufferAttribute(sizes, 1))

  const vertexShader = `
    attribute float size;
    varying vec3 vColor;
    void main() {
      vColor = color;
      vec4 mvPosition = modelViewMatrix * vec4(position, 1.0);
      gl_PointSize = size * (300.0 / -mvPosition.z);
      gl_Position = projectionMatrix * mvPosition;
    }
  `

  const fragmentShader = `
    varying vec3 vColor;
    void main() {
      float dist = length(gl_PointCoord - vec2(0.5));
      if (dist > 0.5) discard;
      float alpha = 1.0 - smoothstep(0.3, 0.5, dist);
      gl_FragColor = vec4(vColor, alpha);
    }
  `

  const material = new THREE.ShaderMaterial({
    vertexShader,
    fragmentShader,
    vertexColors: true,
    transparent: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false
  })

  particles = new THREE.Points(geometry, material)
  scene.add(particles)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  const time = Date.now() * 0.0001

  particles.rotation.y += 0.002
  particles.rotation.x += 0.001

  particles.position.y = Math.sin(time) * 0.5

  const positions = particles.geometry.attributes.position.array
  for (let i = 0; i < particleCount; i++) {
    const i3 = i * 3
    positions[i3 + 1] += Math.sin(time * 2 + i * 0.01) * 0.002
  }
  particles.geometry.attributes.position.needsUpdate = true

  camera.position.x = Math.sin(time * 0.5) * 2
  camera.position.y = Math.cos(time * 0.3) * 1
  camera.lookAt(particles.position)

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
