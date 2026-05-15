<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'
import * as THREE from 'three'
import gsap from 'gsap'
import { TextPlugin } from 'gsap/TextPlugin'
import { motion, AnimatePresence } from 'motion-v'

gsap.registerPlugin(TextPlugin)

// ── State ──
const isActive = ref(false)
const activePanel = ref(0) // 0=none, 1=code, 2=collab, 3=security, 4=future
const containerRef = ref<HTMLDivElement | null>(null)

const panels = [
  { id: 1, tag: 'CODE', title: 'Code', desc: 'The world\'s largest source code repository. Git at planetary scale.', items: ['Unlimited repositories', 'Branch protection rules', 'Code review & ownership', 'Blame, blame, blame'] },
  { id: 2, tag: 'COLLAB', title: 'Collaboration', desc: 'Millions of developers, one shared vision. Merge ideas into reality.', items: ['Pull Request workflow', 'Issue tracking & Projects', 'Discussions & Community', 'Code search & navigation'] },
  { id: 3, tag: 'SECURE', title: 'Security', desc: 'Ship secure code by default. Vulnerability detection before production.', items: ['Dependabot alerts', 'CodeQL analysis', 'Secret scanning', 'Supply chain security'] },
  { id: 4, tag: 'FUTURE', title: 'Future', desc: 'AI-powered development. The next decade of software, already here.', items: ['GitHub Copilot', 'GitHub Actions', 'Codespaces', 'Copilot Chat'] },
]

const activePanelData = computed(() => panels.find(p => p.id === activePanel.value))

// ── Three.js ──
let scene: THREE.Scene, camera: THREE.PerspectiveCamera, renderer: THREE.WebGLRenderer
let particles: THREE.Points
let mouseX = 0, mouseY = 0
let targetX = 0, targetY = 0
let animId = 0

function initThree() {
  const container = containerRef.value
  if (!container) return

  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.z = 30

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.domElement.style.position = 'fixed'
  renderer.domElement.style.inset = '0'
  renderer.domElement.style.zIndex = '0'
  renderer.domElement.style.pointerEvents = 'none'
  container.appendChild(renderer.domElement)

  // Particle system
  const count = 3000
  const positions = new Float32Array(count * 3)
  const colors = new Float32Array(count * 3)
  const sizes = new Float32Array(count)

  for (let i = 0; i < count; i++) {
    // Spiral galaxy-like distribution
    const radius = 8 + Math.random() * 40
    const angle = Math.random() * Math.PI * 2
    const height = (Math.random() - 0.5) * 30

    positions[i * 3] = Math.cos(angle) * radius + (Math.random() - 0.5) * 8
    positions[i * 3 + 1] = height
    positions[i * 3 + 2] = Math.sin(angle) * radius - 10

    const brightness = 0.3 + Math.random() * 0.7
    colors[i * 3] = brightness * 0.7
    colors[i * 3 + 1] = brightness * 0.85
    colors[i * 3 + 2] = brightness

    sizes[i] = Math.random() * 3 + 0.5
  }

  const geo = new THREE.BufferGeometry()
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geo.setAttribute('color', new THREE.BufferAttribute(colors, 3))
  geo.setAttribute('size', new THREE.BufferAttribute(sizes, 1))

  const mat = new THREE.PointsMaterial({
    size: 0.08,
    vertexColors: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
    transparent: true,
    opacity: 0.85,
  })

  particles = new THREE.Points(geo, mat)
  scene.add(particles)

  // Ambient + subtle central light glow (fake via camera position)
  const ambientLight = new THREE.AmbientLight(0x222244, 0.5)
  scene.add(ambientLight)

  window.addEventListener('resize', onResize)
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

function onMouseMove(e: MouseEvent) {
  targetX = (e.clientX / window.innerWidth) * 2 - 1
  targetY = -(e.clientY / window.innerHeight) * 2 + 1
}

function animate() {
  animId = requestAnimationFrame(animate)

  // Smooth mouse follow
  mouseX += (targetX - mouseX) * 0.03
  mouseY += (targetY - mouseY) * 0.03

  if (particles) {
    particles.rotation.y += 0.0002
    particles.rotation.x += (mouseY * 0.15 - particles.rotation.x) * 0.02
    particles.rotation.z += (mouseX * 0.1 - particles.rotation.z) * 0.02
  }

  renderer.render(scene, camera)
}

// ── GSAP Animations ──

function activate() {
  if (isActive.value) return
  isActive.value = true

  const tl = gsap.timeline({ defaults: { ease: 'power4.out', duration: 0.7 } })

  tl.to('.title-main', { x: '-30%', y: '-40%', scale: 0.75 }, 0)
  tl.to('.hero-ring', { scale: 0.5, opacity: 0.2, duration: 0.5 }, 0)
  tl.fromTo('.function-dots', { opacity: 0, scale: 0 }, { opacity: 1, scale: 1, stagger: 0.06, duration: 0.5 }, 0.15)
  tl.fromTo('.panel-glass', { x: '120%', opacity: 0 }, { x: '0%', opacity: 1, duration: 0.65 }, 0.2)
  tl.fromTo('.panel__tag', { y: 30, opacity: 0 }, { y: 0, opacity: 1, duration: 0.4 }, 0.5)
  tl.fromTo('.panel__title', { y: 30, opacity: 0 }, { y: 0, opacity: 1, duration: 0.4 }, 0.55)
  tl.fromTo('.panel__desc', { y: 30, opacity: 0 }, { y: 0, opacity: 1, duration: 0.4 }, 0.6)
  tl.fromTo('.panel__item', { y: 20, opacity: 0 }, { y: 0, opacity: 1, stagger: 0.06, duration: 0.35 }, 0.7)
}

function switchPanel(id: number) {
  activePanel.value = id
  const tl = gsap.timeline({ defaults: { ease: 'power4.out', duration: 0.35 } })
  tl.to('.panel__tag, .panel__title, .panel__desc, .panel__item', { y: -15, opacity: 0, stagger: 0.02, duration: 0.2 })
  tl.fromTo('.panel__tag', { y: 20, opacity: 0 }, { y: 0, opacity: 1, duration: 0.35 }, 0.25)
  tl.fromTo('.panel__title', { y: 20, opacity: 0 }, { y: 0, opacity: 1, duration: 0.35 }, 0.3)
  tl.fromTo('.panel__desc', { y: 20, opacity: 0 }, { y: 0, opacity: 1, duration: 0.35 }, 0.35)
  tl.fromTo('.panel__item', { y: 20, opacity: 0 }, { y: 0, opacity: 1, stagger: 0.05, duration: 0.3 }, 0.4)
}

function deactivate() {
  if (!isActive.value) return
  const tl = gsap.timeline({
    defaults: { ease: 'power4.in', duration: 0.5 },
    onComplete: () => { isActive.value = false; activePanel.value = 0 },
  })
  tl.to('.panel-glass', { x: '120%', opacity: 0, duration: 0.4 })
  tl.to('.function-dots', { opacity: 0, scale: 0.8, stagger: 0.03, duration: 0.3 }, 0)
  tl.to('.title-main', { x: '0%', y: '0%', scale: 1, duration: 0.6 }, 0.1)
  tl.to('.hero-ring', { scale: 1, opacity: 1, duration: 0.5 }, 0.15)
}

// ── Lifecycle ──
onMounted(() => {
  initThree()
  animate()
  window.addEventListener('mousemove', onMouseMove)

  // Entrance animation
  gsap.fromTo('.hero-ring', { scale: 0, opacity: 0 }, { scale: 1, opacity: 1, duration: 1.2, ease: 'elastic.out(1, 0.6)', delay: 0.3 })
  gsap.fromTo('.title-main', { y: 60, opacity: 0 }, { y: 0, opacity: 1, duration: 1, ease: 'power4.out', delay: 0.5 })
  gsap.fromTo('.subtitle', { y: 30, opacity: 0 }, { y: 0, opacity: 1, duration: 0.8, ease: 'power4.out', delay: 0.7 })
})

onBeforeUnmount(() => {
  cancelAnimationFrame(animId)
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('resize', onResize)
  renderer?.dispose()
  particles?.geometry?.dispose()
  ;(particles?.material as THREE.Material)?.dispose()
})
</script>

<template>
  <div ref="containerRef" class="stage">

    <!-- ═══════════════ CENTER RING ═══════════════ -->
    <motion.button
      class="hero-ring"
      :whileHover="{ scale: 1.08, boxShadow: '0 0 60px rgba(255,255,255,0.15)' }"
      :whileTap="{ scale: 0.92 }"
      @click="isActive ? deactivate() : activate()"
    >
      <span class="hero-ring__inner" />
    </motion.button>

    <!-- ═══════════════ MAIN TITLE ═══════════════ -->
    <div class="title-main">
      <h1 class="title-main__text">GITHUB</h1>
      <p class="subtitle">The Digital Cathedral of Code</p>
    </div>

    <!-- ═══════════════ 4 FUNCTION DOTS ═══════════════ -->
    <template v-if="isActive">
      <motion.button
        v-for="dot in panels"
        :key="dot.id"
        :class="['func-dot', `func-dot--${dot.id}`, { 'func-dot--active': activePanel === dot.id }]"
        :whileHover="{ scale: 1.15, boxShadow: '0 0 30px rgba(88,166,255,0.4)' }"
        :whileTap="{ scale: 0.9 }"
        @click="switchPanel(dot.id)"
      >
        <span class="func-dot__icon">
          <template v-if="dot.id === 1">&lt;/&gt;</template>
          <template v-else-if="dot.id === 2">⑂</template>
          <template v-else-if="dot.id === 3">⬢</template>
          <template v-else>✦</template>
        </span>
        <span class="func-dot__label">{{ dot.tag }}</span>
      </motion.button>
    </template>

    <!-- ═══════════════ FROSTED GLASS PANEL ═══════════════ -->
    <AnimatePresence>
      <div v-if="isActive && activePanelData" class="panel-glass" key="panel">
        <div class="panel__tag">{{ activePanelData.tag }}</div>
        <h2 class="panel__title">{{ activePanelData.title }}</h2>
        <p class="panel__desc">{{ activePanelData.desc }}</p>
        <ul class="panel__list">
          <li v-for="item in activePanelData.items" :key="item" class="panel__item">
            <span class="panel__item-marker" />
            {{ item }}
          </li>
        </ul>
      </div>
    </AnimatePresence>

  </div>
</template>

<style scoped>
/* ═══════════════════════════════ STAGE ═══════════════════════════════ */
.stage {
  position: fixed; inset: 0;
  display: flex; align-items: center; justify-content: center;
  overflow: hidden;
  background: #050505;
}

/* ═══════════════════════════════ RING ═══════════════════════════════ */
.hero-ring {
  position: absolute; z-index: 10;
  width: 120px; height: 120px;
  border-radius: 50%;
  background: transparent;
  border: 2px solid rgba(255,255,255,0.15);
  cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  transition: border-color 0.4s, box-shadow 0.4s;
}
.hero-ring:hover {
  border-color: rgba(255,255,255,0.5);
  box-shadow: 0 0 40px rgba(255,255,255,0.1);
}
.hero-ring__inner {
  width: 14px; height: 14px; border-radius: 50%;
  background: rgba(46,164,79,0.7);
  box-shadow: 0 0 20px rgba(46,164,79,0.4);
  animation: pulse-ring 2.5s ease-in-out infinite;
}
@keyframes pulse-ring {
  0%, 100% { transform: scale(1); opacity: 0.7; }
  50% { transform: scale(1.6); opacity: 1; }
}

/* ═══════════════════════════════ TITLE ═══════════════════════════════ */
.title-main {
  position: absolute; z-index: 5;
  text-align: center;
  pointer-events: none;
}
.title-main__text {
  font-size: clamp(4rem, 10vw, 9rem);
  font-weight: 900;
  letter-spacing: -0.04em;
  line-height: 0.9;
  margin: 0;
  color: transparent;
  -webkit-text-stroke: 1.5px rgba(255,255,255,0.25);
  text-stroke: 1.5px rgba(255,255,255,0.25);
  transition: color 0.6s ease, -webkit-text-stroke 0.6s ease;
}
.hero-ring:hover ~ .title-main .title-main__text,
.title-main:hover .title-main__text {
  color: rgba(255,255,255,0.9);
  -webkit-text-stroke: 1.5px rgba(255,255,255,0.6);
}

.subtitle {
  font-size: 0.85rem; font-weight: 400;
  letter-spacing: 0.22em; color: #5a5a62;
  margin-top: 1rem;
}

/* ═══════════════════════════════ FUNCTION DOTS ══════════════════════ */
.func-dot {
  position: absolute; z-index: 15;
  display: flex; flex-direction: column; align-items: center;
  gap: 8px;
  background: transparent; border: none;
  cursor: pointer; padding: 0;
  color: #8a8a82;
  transition: color 0.3s;
}
.func-dot:hover,
.func-dot--active {
  color: #f0f0f0;
}
.func-dot__icon {
  width: 48px; height: 48px;
  border-radius: 50%;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  display: flex; align-items: center; justify-content: center;
  font-size: 1rem;
  transition: border-color 0.3s, background 0.3s;
}
.func-dot:hover .func-dot__icon,
.func-dot--active .func-dot__icon {
  border-color: rgba(88,166,255,0.4);
  background: rgba(88,166,255,0.08);
}
.func-dot__label {
  font-size: 0.65rem; font-weight: 600;
  letter-spacing: 0.18em;
}

.func-dot--1 { top: 12%; right: 28%; }
.func-dot--2 { top: 72%; right: 24%; }
.func-dot--3 { top: 72%; left: 24%; }
.func-dot--4 { top: 12%; left: 28%; }

/* ═══════════════════════════════ FROSTED PANEL ═════════════════════ */
.panel-glass {
  position: absolute; z-index: 20;
  right: 4vw; top: 50%; transform: translateY(-50%);
  width: min(380px, 85vw);
  padding: 2.5rem 2rem;
  background: rgba(255,255,255,0.025);
  backdrop-filter: blur(60px) saturate(140%);
  -webkit-backdrop-filter: blur(60px) saturate(140%);
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: 2px;
}
.panel__tag {
  font-size: 0.65rem; font-weight: 700;
  letter-spacing: 0.28em; color: #2ea44f;
  margin-bottom: 0.75rem;
}
.panel__title {
  font-size: 2.6rem; font-weight: 200;
  letter-spacing: -0.02em; color: #f0f0f0;
  margin: 0 0 1rem; line-height: 1;
}
.panel__desc {
  font-size: 0.9rem; line-height: 1.6;
  color: #8a8a82; margin: 0 0 2rem;
}
.panel__list {
  list-style: none; padding: 0; margin: 0;
  display: flex; flex-direction: column; gap: 12px;
}
.panel__item {
  font-size: 0.85rem; color: #d0d0d4;
  display: flex; align-items: center; gap: 12px;
}
.panel__item-marker {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: rgba(88,166,255,0.35);
  flex-shrink: 0;
}
</style>
