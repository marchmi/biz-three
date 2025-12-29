# Three.js 快速入门指南

## 1. Three.js 简介

Three.js 是一个基于 WebGL 的 JavaScript 3D 库，它封装了 WebGL 的底层 API，使开发者能够更轻松地创建 3D 图形。

**主要特点：**
- 跨浏览器兼容
- 丰富的几何体、材质、光照支持
- 强大的动画系统
- 支持 Shader 编程
- 活跃的社区和生态系统

## 2. 安装与引入

### 2.1 npm 安装（推荐）

```bash
npm install three
```

在代码中引入：
```javascript
import * as THREE from 'three'
```

### 2.2 CDN 引入

```html
<script type="importmap">
  {
    "imports": {
      "three": "https://unpkg.com/three@0.160.0/build/three.module.js"
    }
  }
</script>
```

## 3. Three.js 核心概念

### 3.1 场景（Scene）

场景是所有 3D 对象的容器，类似于一个虚拟的三维空间。

```javascript
const scene = new THREE.Scene()
scene.background = new THREE.Color(0x1a1a2e)  // 设置背景颜色
```

### 3.2 相机（Camera）

相机决定了观察场景的视角。

**透视相机（PerspectiveCamera）：**
```javascript
const camera = new THREE.PerspectiveCamera(
  75,                              // 视野角度
  width / height,                  // 宽高比
  0.1,                             // 近裁剪面
  1000                             // 远裁剪面
)
camera.position.z = 5              // 设置相机位置
```

### 3.3 渲染器（Renderer）

负责将场景渲染到 Canvas 上。

```javascript
const renderer = new THREE.WebGLRenderer({ antialias: true })
renderer.setSize(width, height)
renderer.setPixelRatio(window.devicePixelRatio)
document.body.appendChild(renderer.domElement)
```

### 3.4 几何体（Geometry）

Three.js 提供了丰富的几何体：

```javascript
// 立方体
const boxGeometry = new THREE.BoxGeometry(width, height, depth)

// 球体
const sphereGeometry = new THREE.SphereGeometry(radius, widthSegments, heightSegments)

// 环形结
const torusKnotGeometry = new THREE.TorusKnotGeometry(radius, tube, tubularSegments, radialSegments)
```

### 3.5 材质（Material）

材质决定了物体的外观。

**MeshBasicMaterial（基础材质）：**
```javascript
const material = new THREE.MeshBasicMaterial({
  color: 0x667eea
})
```

**MeshLambertMaterial（朗伯材质）：**
```javascript
const material = new THREE.MeshLambertMaterial({
  color: 0x4ecdc4
})
```

**MeshPhongMaterial（冯氏材质）：**
```javascript
const material = new THREE.MeshPhongMaterial({
  color: 0xffe66d,
  shininess: 100        // 高光光泽度
})
```

**MeshStandardMaterial（标准材质）：**
```javascript
const material = new THREE.MeshStandardMaterial({
  color: 0x95e1d3,
  roughness: 0.3,       // 粗糙度
  metalness: 0.2        // 金属度
})
```

### 3.6 网格（Mesh）

网格是几何体和材质的结合。

```javascript
const geometry = new THREE.BoxGeometry(1, 1, 1)
const material = new THREE.MeshPhongMaterial({ color: 0x667eea })
const mesh = new THREE.Mesh(geometry, material)
scene.add(mesh)
```

### 3.7 光照（Light）

**环境光（AmbientLight）：**
```javascript
const ambientLight = new THREE.AmbientLight(0x404040, 0.5)
scene.add(ambientLight)
```

**平行光（DirectionalLight）：**
```javascript
const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
directionalLight.position.set(5, 5, 5)
scene.add(directionalLight)
```

**点光源（PointLight）：**
```javascript
const pointLight = new THREE.PointLight(0xff6b6b, 2, 20)
pointLight.position.set(0, 0, 5)
scene.add(pointLight)
```

## 4. 动画循环

使用 `requestAnimationFrame` 创建平滑的动画：

```javascript
function animate() {
  requestAnimationFrame(animate)
  
  mesh.rotation.x += 0.01
  mesh.rotation.y += 0.01
  
  renderer.render(scene, camera)
}
animate()
```

## 5. 响应式设计

监听窗口大小变化：

```javascript
function handleResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}
window.addEventListener('resize', handleResize)
```

## 6. 清理资源

组件卸载时清理资源：

```javascript
onBeforeUnmount(() => {
  cancelAnimationFrame(animationId)
  window.removeEventListener('resize', handleResize)
  
  // 释放几何体和材质
  geometry.dispose()
  material.dispose()
  
  // 释放渲染器
  renderer.dispose()
})
```

## 7. 常用技巧

### 7.1 使用辅助工具

```javascript
// 坐标轴辅助
const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)

// 网格辅助
const gridHelper = new THREE.GridHelper(10, 10)
scene.add(gridHelper)

// 光源辅助
const pointLightHelper = new THREE.PointLightHelper(pointLight, 0.5)
scene.add(pointLightHelper)
```

### 7.2 阴影效果

```javascript
renderer.shadowMap.enabled = true
directionalLight.castShadow = true
mesh.castShadow = true
mesh.receiveShadow = true
```

### 7.3 自定义 Shader

```javascript
const vertexShader = `
  void main() {
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0)
  }
`

const fragmentShader = `
  void main() {
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0)
  }
`

const material = new THREE.ShaderMaterial({
  vertexShader,
  fragmentShader
})
```

## 8. 性能优化建议

1. **复用几何体和材质**
2. **使用 BufferGeometry 代替 Geometry**
3. **合理控制粒子数量**
4. **及时清理不需要的资源**
5. **使用 InstancedMesh 渲染大量相同物体**
6. **开启渲染器的像素比限制**

```javascript
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
```

## 9. 学习资源

- [Three.js 官方文档](https://threejs.org/)
- [Three.js GitHub](https://github.com/mrdoob/three.js)
- [Three.js 示例](https://threejs.org/examples/)
- [Three.js 在线编辑器](https://threejs.org/editor/)
