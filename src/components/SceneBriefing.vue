<template>
  <div class="scene briefing-wrap">

    <!-- ══ 雷暴序章 ══ -->
    <Transition name="storm-exit">
      <div v-if="phase === 'storm'" class="storm-screen" @click="startBriefing">

        <!-- 背景（随闪电改变） -->
        <div class="storm-bg"></div>

        <!-- 雨丝 -->
        <div class="rain-layer"></div>

        <!-- 闪电 SVG -->
        <svg class="bolt-svg" viewBox="0 0 420 500" preserveAspectRatio="xMidYMid meet">
          <path class="bolt bolt-l"
                d="M 128 30 L 100 200 L 120 200 L 86 390 L 162 225 L 140 225 Z"
                fill="#cce8ff" filter="url(#bolt-glow)"/>
          <path class="bolt bolt-r"
                d="M 308 10 L 278 175 L 298 175 L 260 355 L 338 195 L 316 195 Z"
                fill="#ddf2ff" filter="url(#bolt-glow)"/>
          <defs>
            <filter id="bolt-glow" x="-60%" y="-20%" width="220%" height="140%">
              <feGaussianBlur stdDeviation="6" result="blur"/>
              <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
            </filter>
          </defs>
        </svg>

        <!-- 中央文字（随闪电渐现） -->
        <div class="storm-center">
          <div class="storm-stamp">机密档案</div>
          <div class="storm-title">幸存者法庭</div>
          <div class="storm-sub">SURVIVOR'S COURT · SRG · 1943</div>
        </div>

        <div class="storm-skip">点击任意处跳过</div>
      </div>
    </Transition>

    <!-- ══ 正文简报 ══ -->
    <template v-if="phase === 'briefing'">
      <div style="margin-bottom:32px" class="fade-in">
        <div class="stamp" style="margin-bottom:16px">任务简报</div>
        <h1 style="margin-bottom:8px">幸存者法庭</h1>
        <p style="color:var(--amber-dim); font-size:0.95rem; letter-spacing:0.18em; font-family:var(--font-display)">
          SURVIVOR'S COURT · STATISTICAL RESEARCH GROUP · 1943
        </p>
      </div>

      <div class="card fade-in" style="margin-bottom:24px">
        <p style="font-size:1.12rem; line-height:2.1" v-html="displayedText"></p>
        <span v-if="!typingDone" class="cursor"> </span>
        <button v-if="!typingDone" class="skip-btn" @click="skipTyping">快速跳过 ▸</button>
      </div>

      <div v-if="typingDone" class="fade-in" style="margin-bottom:32px">
        <PlaneMap :show-bullet-holes="true" :interactive="false" />
        <p style="text-align:center; margin-top:12px; font-size:0.8rem; color:var(--amber-dim); letter-spacing:0.1em">
          ↑ 返航飞机弹孔分布图（综合 87 架次数据）
        </p>
      </div>

      <div v-if="typingDone" class="fade-in" style="display:flex; justify-content:center">
        <button class="btn" @click="$emit('next')">开始调查 →</button>
      </div>
    </template>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import PlaneMap from './PlaneMap.vue'

defineEmits(['next'])

const phase = ref('storm')
const displayedText = ref('')
const typingDone = ref(false)
let intervalId = null

// 序章自动推进（3.2s）
let stormTimer = null
onMounted(() => {
  stormTimer = setTimeout(startBriefing, 3200)
})

function startBriefing() {
  if (phase.value !== 'storm') return
  clearTimeout(stormTimer)
  phase.value = 'briefing'
  // 短暂延迟后再开打字机，等 fade-in 完成
  setTimeout(startTyping, 120)
}

const briefingText =
`华盛顿，1943年。<br><br>
美军第八航空队的B-17轰炸机正在欧洲上空执行任务。<br>
每一次出击，都有飞机一去不回。<br><br>
军方工程师收集了所有<strong>返航飞机</strong>的弹孔数据，<br>
绘制成下方这张分布图。<br><br>
指挥官找到你——数据分析师——只问一个问题：<br><br>
<strong>「我们应该在飞机的哪里加装甲？」</strong><br><br>
在做出判断之前，你可以审讯三位证人。`

function startTyping() {
  let i = 0
  intervalId = setInterval(() => {
    if (i >= briefingText.length) {
      clearInterval(intervalId)
      typingDone.value = true
      return
    }
    if (briefingText[i] === '<') {
      const close = briefingText.indexOf('>', i)
      displayedText.value = briefingText.slice(0, close + 1)
      i = close + 1
    } else {
      i++
      displayedText.value = briefingText.slice(0, i)
    }
  }, 22)
}

function skipTyping() {
  if (intervalId) clearInterval(intervalId)
  displayedText.value = briefingText
  typingDone.value = true
}
</script>

<style scoped>
.briefing-wrap { position: relative; min-height: 400px; }

/* ══ 雷暴屏 ══ */
.storm-screen {
  position: absolute;
  inset: -72px -48px;          /* 撑满 .scene 的内边距 */
  z-index: 10;
  cursor: pointer;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 背景随闪电颜色变化 */
.storm-bg {
  position: absolute;
  inset: 0;
  background: #020608;
  animation: stormBg 3.2s ease-out forwards;
}
@keyframes stormBg {
  0%,  100% { background: #020608; }
  15%        { background: #0d2035; }
  20%        { background: #060e18; }
  44%        { background: #0a1828; }
  47%        { background: #040c16; }
  72%        { background: #102040; }
  79%        { background: #060e1a; }
}

/* 雨丝（CSS 纹理动画） */
.rain-layer {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background-image: repeating-linear-gradient(
    172deg,
    transparent 0px, transparent 5px,
    rgba(160,210,255,0.10) 5px, rgba(160,210,255,0.10) 6px
  );
  background-size: 8px 8px;
  animation: rainFall 0.09s linear infinite;
  opacity: 0.7;
}
@keyframes rainFall {
  from { background-position: 0 0; }
  to   { background-position: 8px 8px; }
}

/* 闪电 SVG */
.bolt-svg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}
.bolt {
  opacity: 0;
  animation: boltFlash 3.2s ease-out forwards;
}
.bolt-l { animation-delay: 0ms; }
.bolt-r { animation-delay: 60ms; transform-origin: center; }

@keyframes boltFlash {
  0%,  11%, 24%, 39%, 52%, 67%, 84%, 100% { opacity: 0; }
  /* 第一击 */
  13% { opacity: 0.4; }
  15% { opacity: 1; }
  18% { opacity: 0.15; }
  20% { opacity: 0.95; }
  23% { opacity: 0; }
  /* 第二击 */
  41% { opacity: 0.7; }
  44% { opacity: 0.2; }
  47% { opacity: 0.9; }
  51% { opacity: 0; }
  /* 第三击（最强） */
  69% { opacity: 0.5; }
  72% { opacity: 1; }
  75% { opacity: 0.5; }
  78% { opacity: 1; }
  83% { opacity: 0; }
}

/* 中央文字 */
.storm-center {
  position: relative;
  z-index: 2;
  text-align: center;
  animation: titleReveal 3.2s ease-out forwards;
}
@keyframes titleReveal {
  0%, 12%, 100% { opacity: 0; }
  15%, 22%      { opacity: 1; }
  23%           { opacity: 0; }
  41%, 51%      { opacity: 0.85; }
  52%           { opacity: 0.05; }
  69%, 83%      { opacity: 1; }
  84%, 95%      { opacity: 0.15; }
  97%           { opacity: 0; }
}

.storm-stamp {
  display: inline-block;
  border: 3px solid #c0392b;
  color: #c0392b;
  font-family: var(--font-display);
  font-size: 0.9rem;
  letter-spacing: 0.35em;
  padding: 4px 18px;
  margin-bottom: 20px;
  text-shadow: 0 0 12px rgba(192,57,43,0.9);
  box-shadow: 0 0 14px rgba(192,57,43,0.4);
  transform: rotate(-6deg);
}
.storm-title {
  font-family: var(--font-display);
  font-size: 2.8rem;
  letter-spacing: 0.22em;
  color: #cce8ff;
  text-shadow: 0 0 20px rgba(160,220,255,0.9), 0 0 60px rgba(100,180,255,0.4);
  margin-bottom: 10px;
}
.storm-sub {
  font-family: var(--font-mono);
  font-size: 0.72rem;
  letter-spacing: 0.25em;
  color: rgba(160,210,255,0.6);
}

.storm-skip {
  position: absolute;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  font-family: var(--font-mono);
  font-size: 0.72rem;
  color: rgba(160,210,255,0.35);
  letter-spacing: 0.15em;
  animation: skipPulse 1.8s ease-in-out infinite;
}
@keyframes skipPulse {
  0%, 100% { opacity: 0.3; }
  50%       { opacity: 0.7; }
}

/* 序章淡出 */
.storm-exit-leave-active { transition: opacity 0.4s ease, transform 0.4s ease; }
.storm-exit-leave-to     { opacity: 0; transform: scale(1.04); }

/* ── 打字光标 ── */
.cursor {
  display: inline-block;
  width: 2px;
  height: 1.1em;
  background: var(--amber);
  vertical-align: text-bottom;
  margin-left: 2px;
  animation: blink 0.9s step-end infinite;
}
@keyframes blink { 50% { opacity: 0; } }

/* ── 跳过按钮 ── */
.skip-btn {
  display: block;
  background: transparent;
  border: none;
  color: var(--amber-dim);
  font-family: var(--font-mono);
  font-size: 0.78rem;
  cursor: pointer;
  margin-top: 16px;
  padding: 4px 8px;
  opacity: 0.6;
  transition: opacity 0.2s, color 0.2s;
  letter-spacing: 0.05em;
}
.skip-btn:hover { opacity: 1; color: var(--amber); }
</style>
