<template>
  <div class="app">
    <header class="header">
      <h1>Three.js 可视化示例</h1>
      <p class="subtitle">Vue3 + JavaScript + Three.js</p>
    </header>
    
    <main class="main-content">
      <nav class="sidebar">
        <h2>示例列表</h2>
        <ul>
          <li 
            v-for="example in examples" 
            :key="example.id"
            :class="{ active: currentExample === example.id }"
            @click="currentExample = example.id"
          >
            {{ example.name }}
          </li>
        </ul>
      </nav>
      
      <div class="content">
        <component :is="currentComponent" />
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import BasicScene from './components/BasicScene.vue'
import CubeExample from './components/CubeExample.vue'
import SphereExample from './components/SphereExample.vue'
import TorusKnotExample from './components/TorusKnotExample.vue'
import ParticlesExample from './components/ParticlesExample.vue'

const examples = [
  { id: 'basic', name: '基础场景' },
  { id: 'cube', name: '旋转立方体' },
  { id: 'sphere', name: '球体与光照' },
  { id: 'torus', name: '环形结' },
  { id: 'particles', name: '粒子系统' }
]

const currentExample = ref('basic')

const currentComponent = computed(() => {
  const components = {
    basic: BasicScene,
    cube: CubeExample,
    sphere: SphereExample,
    torus: TorusKnotExample,
    particles: ParticlesExample
  }
  return components[currentExample.value]
})
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px 40px;
  text-align: center;
}

.header h1 {
  font-size: 2rem;
  margin-bottom: 5px;
}

.subtitle {
  font-size: 0.9rem;
  opacity: 0.8;
}

.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.sidebar {
  width: 200px;
  background: #2d2d44;
  padding: 20px;
  overflow-y: auto;
}

.sidebar h2 {
  color: #fff;
  font-size: 1.1rem;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 2px solid #667eea;
}

.sidebar ul {
  list-style: none;
}

.sidebar li {
  color: #ccc;
  padding: 12px 15px;
  margin: 5px 0;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.sidebar li:hover {
  background: rgba(102, 126, 234, 0.3);
  color: #fff;
}

.sidebar li.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
}

.content {
  flex: 1;
  background: #16213e;
}
</style>
