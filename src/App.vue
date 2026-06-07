<template>
  <!-- CRT 开机闪烁遮罩 -->
  <div class="crt-boot" :class="{ done: bootDone }" aria-hidden="true" />

  <div class="top-bar">
    <span class="stamp">机密</span>
    <span>陆军航空队 · 统计分析部门 · 1943</span>
    <span style="margin-left:auto">档案编号 SRG-43-{{ missionId }}</span>
    <button
      class="theme-toggle"
      @click="toggleTheme"
      :title="isDark ? '切换至档案纸模式（浅色）' : '切换至夜视模式（深色）'"
    >
      <!-- 深色模式：显示月亮（当前状态）-->
      <svg v-if="isDark" class="theme-icon" viewBox="0 0 48 48" width="20" height="20">
        <path d="M32 8 A 16 16 0 1 1 8 32 A 12 12 0 1 0 32 8"
              fill="currentColor" stroke="currentColor" stroke-width="1.2" stroke-linejoin="round"/>
        <circle cx="18" cy="18" r="1.2" fill="none" stroke="currentColor" stroke-width="0.8" opacity="0.5"/>
        <circle cx="24" cy="14" r="0.8" fill="none" stroke="currentColor" stroke-width="0.6" opacity="0.4"/>
        <circle cx="20" cy="26" r="1"   fill="none" stroke="currentColor" stroke-width="0.7" opacity="0.45"/>
      </svg>
      <!-- 浅色模式：显示太阳（当前状态）-->
      <svg v-else class="theme-icon" viewBox="0 0 48 48" width="20" height="20">
        <circle cx="24" cy="24" r="18" fill="none" stroke="currentColor" stroke-width="1.2" stroke-dasharray="4 3"/>
        <circle cx="24" cy="24" r="9"  fill="none" stroke="currentColor" stroke-width="1.8"/>
        <g stroke="currentColor" stroke-width="1.5" stroke-linecap="round">
          <line x1="24" y1="2"  x2="24" y2="7"/>
          <line x1="24" y1="41" x2="24" y2="46"/>
          <line x1="2"  y1="24" x2="7"  y2="24"/>
          <line x1="41" y1="24" x2="46" y2="24"/>
          <line x1="8.5"  y1="8.5"  x2="12" y2="12"/>
          <line x1="36"   y1="36"   x2="39.5" y2="39.5"/>
          <line x1="8.5"  y1="39.5" x2="12" y2="36"/>
          <line x1="36"   y1="12"   x2="39.5" y2="8.5"/>
        </g>
        <circle cx="24" cy="24" r="2" fill="currentColor"/>
      </svg>
    </button>
  </div>

  <!-- 场景切换：滑动过渡 -->
  <Transition :name="transitionName" mode="out-in">
    <component
      :is="currentScene"
      :key="sceneIndex"
      v-if="sceneIndex === 4"
      :latestChoice="lastChoice"
      :decisionHistory="decisionHistory"
      @next="goNext"
      @retry="goToRetry"
    />
    <component
      :is="currentScene"
      :key="sceneIndex"
      v-else
      @next="sceneIndex === 3 ? goToParallel() : goNext()"
      @verdict="handleVerdict"
    />
  </Transition>
</template>

<script setup>
import { ref, computed, provide, onMounted } from 'vue'
import SceneBriefing from './components/SceneBriefing.vue'
import SceneInterrogation from './components/SceneInterrogation.vue'
import SceneVerdict from './components/SceneVerdict.vue'
import SceneRevelation from './components/SceneRevelation.vue'
import SceneParallel from './components/SceneParallel.vue'

const scenes = [SceneBriefing, SceneInterrogation, SceneVerdict, SceneRevelation, SceneParallel]
const sceneIndex = ref(0)
const verdictCorrect = ref(false)
const verdictType = ref('wrong')
const missionId = Math.floor(Math.random() * 9000 + 1000)
const decisionHistory = ref([])
const lastChoice = ref(new Set())
const bootDone = ref(false)
const transitionName = ref('slide-forward')

// ── 主题（读取 localStorage 偏好，默认深色）──
const savedTheme = localStorage.getItem('theme')
const isDark = ref(savedTheme ? savedTheme === 'dark' : true)

function applyTheme(dark) {
  document.documentElement.setAttribute('data-theme', dark ? 'dark' : 'light')
  localStorage.setItem('theme', dark ? 'dark' : 'light')
}

function toggleTheme() {
  isDark.value = !isDark.value
  applyTheme(isDark.value)
}

const currentScene = computed(() => scenes[sceneIndex.value])

onMounted(() => {
  applyTheme(isDark.value)   // 确保刷新后立即应用
  setTimeout(() => { bootDone.value = true }, 600)
})

function goNext() {
  transitionName.value = 'slide-forward'
  if (sceneIndex.value < scenes.length - 1) sceneIndex.value++
}

function handleVerdict({ correct, verdictType: vt, zones }) {
  transitionName.value = 'slide-forward'
  verdictCorrect.value = correct
  verdictType.value = vt ?? 'wrong'
  
  // 记录这次决策
  lastChoice.value = new Set(zones)
  decisionHistory.value.push({
    choice: zones,
    type: vt,
    timestamp: Date.now()
  })
  
  // 跳转到真相页
  sceneIndex.value = 3
}

function goToParallel() {
  transitionName.value = 'slide-forward'
  sceneIndex.value = 4
}

function goToRetry() {
  transitionName.value = 'slide-backward'
  sceneIndex.value = 2
}

provide('verdictCorrect', verdictCorrect)
provide('verdictType', verdictType)
</script>

<style scoped>
/* ── CRT 开机闪烁 ── */
.crt-boot {
  position: fixed;
  inset: 0;
  background: #000;
  z-index: 99999;
  animation: crtBoot 600ms steps(1, end) forwards;
  pointer-events: none;
}
.crt-boot.done { display: none; }

@keyframes crtBoot {
  0%   { opacity: 1; background: #000; }
  15%  { opacity: 1; background: #0a1a0a; }
  28%  { opacity: 0; }
  40%  { opacity: 1; background: #000; }
  55%  { opacity: 0; }
  68%  { opacity: 1; background: #040d04; }
  82%  { opacity: 0; }
  92%  { opacity: 0.12; }
  100% { opacity: 0; }
}

/* ── 场景切换 ── */
.slide-forward-enter-active,
.slide-forward-leave-active {
  transition: opacity 0.45s ease-out, transform 0.5s ease-out;
}
.slide-forward-enter-from {
  opacity: 0;
  transform: translateX(32px);
}
.slide-forward-leave-to {
  opacity: 0;
  transform: translateX(-24px);
}

/* 滑动返回（向过去回溯） */
.slide-backward-enter-active,
.slide-backward-leave-active {
  transition: opacity 0.45s ease-out, transform 0.5s ease-out;
}
.slide-backward-enter-from {
  opacity: 0;
  transform: translateX(-32px);
}
.slide-backward-leave-to {
  opacity: 0;
  transform: translateX(24px);
}
</style>
