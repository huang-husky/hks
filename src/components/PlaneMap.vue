<template>
  <div class="plane-map-wrap">
    <svg viewBox="0 0 420 350" xmlns="http://www.w3.org/2000/svg" class="plane-svg">
      <defs>
        <!-- 选择区域高亮渐变 -->
        <radialGradient id="selected-grad" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#c8a84b" stop-opacity="0.35"/>
          <stop offset="100%" stop-color="#c8a84b" stop-opacity="0.08"/>
        </radialGradient>
        <!-- 最优选择高亮 -->
        <radialGradient id="optimal-grad" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#4a8a4a" stop-opacity="0.4"/>
          <stop offset="100%" stop-color="#4a8a4a" stop-opacity="0.1"/>
        </radialGradient>
      </defs>
      <defs>
        <!-- 纸张噪点纹理 -->
        <filter id="paper-texture" x="-5%" y="-5%" width="110%" height="110%">
          <feTurbulence type="fractalNoise" baseFrequency="0.04" numOctaves="5" result="noise"/>
          <feDiffuseLighting in="noise" lighting-color="#f4e4c1" surfaceScale="1.5" result="light">
            <feDistantLight azimuth="45" elevation="60"/>
          </feDiffuseLighting>
          <feBlend in="SourceGraphic" in2="light" mode="multiply"/>
        </filter>

        <!-- 铅笔线条抖动 -->
        <filter id="pencil-wobble" x="-2%" y="-2%" width="104%" height="104%">
          <feTurbulence type="turbulence" baseFrequency="0.025" numOctaves="2" result="noise"/>
          <feDisplacementMap in="SourceGraphic" in2="noise" scale="0.6"
            xChannelSelector="R" yChannelSelector="G"/>
        </filter>

        <!-- 弹孔投影 -->
        <filter id="hole-shadow" x="-60%" y="-60%" width="220%" height="220%">
          <feDropShadow dx="1" dy="1.2" stdDeviation="0.7"
            flood-color="#2a2a3a" flood-opacity="0.45"/>
        </filter>

        <!-- 纸张边缘渐变 -->
        <radialGradient id="paper-edge" cx="50%" cy="50%" r="50%">
          <stop offset="58%" stop-color="#f4e4c1" stop-opacity="0"/>
          <stop offset="100%" stop-color="#b89a60" stop-opacity="0.35"/>
        </radialGradient>

        <!-- 幽灵弹孔渗入渐变 -->
        <radialGradient id="ghost-grad" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#8b0000"/>
          <stop offset="100%" stop-color="#c0392b" stop-opacity="0.6"/>
        </radialGradient>
      </defs>

      <!-- ── 纸张背景 ── -->
      <rect x="0" y="0" width="420" height="350" fill="#f4e4c1" rx="3"/>
      <rect x="0" y="0" width="420" height="350" fill="url(#paper-edge)" rx="3"/>
      <rect x="0" y="0" width="420" height="350" fill="#f4e4c1"
            filter="url(#paper-texture)" opacity="0.28" rx="3"/>

      <!-- ── 蓝图网格 ── -->
      <g stroke="#4a6080" stroke-width="0.4" opacity="0.07">
        <line v-for="n in 10" :key="'gx'+n" :x1="n*42" y1="0" :x2="n*42" y2="350"/>
        <line v-for="n in 9"  :key="'gy'+n" x1="0" :y1="n*39" x2="420" :y2="n*39"/>
      </g>

      <!-- ── CLASSIFIED 水印 ── -->
      <g opacity="0.07" transform="rotate(-15 210 175)">
        <rect x="105" y="148" width="210" height="54" fill="none"
              stroke="#8b0000" stroke-width="2.5"/>
        <text x="210" y="172" text-anchor="middle"
              font-family="'Special Elite', monospace" font-size="22"
              fill="#8b0000" letter-spacing="5">CLASSIFIED</text>
        <text x="210" y="192" text-anchor="middle"
              font-family="'Special Elite', monospace" font-size="8"
              fill="#8b0000" letter-spacing="2">文件编号 SRG-43-0891</text>
      </g>

      <!-- ══ 飞机图片 ══ -->
      <image
        :href="planeImg"
        x="5" y="5"
        width="410" height="335"
        preserveAspectRatio="xMidYMid meet"
        style="mix-blend-mode: multiply"
      />

      <!-- ══ 弹孔（返航飞机，逐个打入）══ -->
      <template v-if="showBulletHoles">
        <g v-for="(h, i) in bulletHoles" :key="'bh'+i"
           class="hole-punch"
           :style="{ animationDelay: h.delay + 'ms' }">
          <circle :cx="h.x" :cy="h.y" r="3.8"
                  fill="#f2e2b8" :stroke="h.rust ?? '#8b4513'"
                  stroke-width="1.5" filter="url(#hole-shadow)"/>
          <circle :cx="h.x" :cy="h.y" r="2.1" fill="#1a1a2a" opacity="0.72"/>
          <path   :d="crackPath(h.x, h.y)"
                  stroke="#8b4513" stroke-width="0.45" fill="none" opacity="0.82"/>
          <circle :cx="h.x" :cy="h.y" r="4.2" fill="none"
                  stroke="#a0522d" stroke-width="0.4" stroke-dasharray="1.2,2.2"/>
        </g>
        <!-- 发动机 / 驾驶舱：极少弹孔（刻意稀少）-->
        <circle cx="90"  cy="173" r="1.8" fill="#1a1a2a" opacity="0.25"/>
        <circle cx="202" cy="88"  r="1.6" fill="#1a1a2a" opacity="0.22"/>
      </template>

      <!-- ══ 幽灵弹孔（未返航飞机，渗入动画）══ -->
      <template v-if="showGhost">
        <g v-for="(h, i) in ghostHoles" :key="'gh'+i"
           class="ghost-bleed"
           :style="{ animationDelay: (i * 90) + 'ms' }">
          <circle :cx="h.x" :cy="h.y" r="4.2"
                  fill="url(#ghost-grad)" stroke="#c0392b"
                  stroke-width="1.3" filter="url(#hole-shadow)"/>
          <circle :cx="h.x" :cy="h.y" r="2"   fill="#1a0000" opacity="0.85"/>
          <path   :d="crackPath(h.x, h.y)"
                  stroke="#c0392b" stroke-width="0.5" fill="none" opacity="0.7"/>
        </g>
      </template>

      <!-- ══ 手写标注 ══ -->
      <g opacity="0.78" font-family="'Shouxie', cursive" fill="#4a6080">
        <!-- 左机翼 -->
        <line x1="90"  y1="169" x2="52"  y2="189" stroke="#4a6080" stroke-width="0.75"/>
        <text x="14"  y="199" font-size="10" transform="rotate(-5 14 199)">左机翼</text>

        <!-- 右机翼 -->
        <line x1="318" y1="169" x2="358" y2="189" stroke="#4a6080" stroke-width="0.75"/>
        <text x="350" y="199" font-size="10" transform="rotate(5 350 199)">右机翼</text>

        <!-- 机身 -->
        <line x1="232" y1="195" x2="270" y2="183" stroke="#4a6080" stroke-width="0.7"/>
        <text x="272"  y="181" font-size="10">机身</text>

        <!-- 左发动机（单独标，指向内侧发动机）-->
        <line x1="148" y1="157" x2="130" y2="207" stroke="#4a6080" stroke-width="0.7"/>
        <text x="62"   y="221" font-size="9">左发动机</text>

        <!-- 右发动机 -->
        <line x1="262" y1="157" x2="278" y2="207" stroke="#4a6080" stroke-width="0.7"/>
        <text x="268"  y="221" font-size="9">右发动机</text>

        <!-- 驾驶舱 -->
        <line x1="222" y1="77"  x2="258" y2="63"  stroke="#4a6080" stroke-width="0.7"/>
        <text x="260"  y="61"   font-size="9">驾驶舱</text>

        <!-- 图例 -->
        <text x="210" y="337" text-anchor="middle"
              font-family="'Special Elite', monospace" font-size="7"
              fill="#6a7a8a" letter-spacing="2">
          FIG.1 — B-17G FLYING FORTRESS · DAMAGE ASSESSMENT · SRG-1943
        </text>
      </g>

      <!-- ══ 选择区域高亮（只显示不交互）══ -->
      <template v-if="showSelectedZones">
        <!-- 左机翼 -->
        <polygon v-if="selectedZones.has('left-wing')"
                 points="190,140 55,160 55,185 190,180"
                 fill="url(#selected-grad)" stroke="#c8a84b" stroke-width="2"
                 stroke-dasharray="4,3" rx="4"/>
        <!-- 右机翼 -->
        <polygon v-if="selectedZones.has('right-wing')"
                 points="218,140 358,160 358,185 218,180"
                 fill="url(#selected-grad)" stroke="#c8a84b" stroke-width="2"
                 stroke-dasharray="4,3" rx="4"/>
        <!-- 机身 -->
        <ellipse v-if="selectedZones.has('fuselage')"
                 cx="205" cy="180" rx="25" ry="95"
                 fill="url(#selected-grad)" stroke="#c8a84b" stroke-width="2"
                 stroke-dasharray="4,3"/>
        <!-- 左发动机 -->
        <rect v-if="selectedZones.has('left-engine')"
              x="72" y="143" width="108" height="44" rx="4"
              fill="url(#selected-grad)" stroke="#c8a84b" stroke-width="2"
              stroke-dasharray="4,3"/>
        <!-- 右发动机 -->
        <rect v-if="selectedZones.has('right-engine')"
              x="228" y="143" width="108" height="44" rx="4"
              fill="url(#selected-grad)" stroke="#c8a84b" stroke-width="2"
              stroke-dasharray="4,3"/>
        <!-- 驾驶舱 -->
        <ellipse v-if="selectedZones.has('cockpit')"
                 cx="205" cy="77" rx="26" ry="34"
                 fill="url(#selected-grad)" stroke="#c8a84b" stroke-width="2"
                 stroke-dasharray="4,3"/>
      </template>

      <!-- ══ 可交互点击区（透明覆盖层，贴合图片飞机）══ -->
      <template v-if="interactive">
        <!-- 左机翼（翼展弧形区域）-->
        <polygon points="190,140 55,160 55,185 190,180"
                 fill="transparent" class="zone"
                 :class="{ selected: selectedZones.has('left-wing') }"
                 @click="selectZone('left-wing')"/>
        <!-- 右机翼 -->
        <polygon points="218,140 358,160 358,185 218,180"
                 fill="transparent" class="zone"
                 :class="{ selected: selectedZones.has('right-wing') }"
                 @click="selectZone('right-wing')"/>
        <!-- 机身 -->
        <ellipse cx="205" cy="180" rx="25" ry="95"
                 fill="transparent" class="zone"
                 :class="{ selected: selectedZones.has('fuselage') }"
                 @click="selectZone('fuselage')"/>
        <!-- 左侧发动机组（两台）-->
        <rect x="72" y="143" width="108" height="44" rx="4"
              fill="transparent" class="zone"
              :class="{ selected: selectedZones.has('left-engine') }"
              @click="selectZone('left-engine')"/>
        <!-- 右侧发动机组（两台）-->
        <rect x="228" y="143" width="108" height="44" rx="4"
              fill="transparent" class="zone"
              :class="{ selected: selectedZones.has('right-engine') }"
              @click="selectZone('right-engine')"/>
        <!-- 驾驶舱 -->
        <ellipse cx="205" cy="77" rx="26" ry="34"
                 fill="transparent" class="zone"
                 :class="{ selected: selectedZones.has('cockpit') }"
                 @click="selectZone('cockpit')"/>
      </template>
    </svg>

    <div v-if="interactive && selectedZones.size > 0" class="zone-label fade-in">
      已标记 {{ selectedZones.size }} 处 · 再次点击可取消
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import planeImg from '../assets/plane.png'

const props = defineProps({
  showBulletHoles: { type: Boolean, default: false },
  showGhost:       { type: Boolean, default: false },
  interactive:     { type: Boolean, default: false },
  selectedZones:   { type: Set, default: () => new Set() },
  showSelectedZones: { type: Boolean, default: false },
})
const emit = defineEmits(['select', 'update:selectedZones'])

const localSelected = reactive(new Set())
const selectedZones = props.showSelectedZones 
  ? props.selectedZones 
  : localSelected
const correctZones = new Set(['left-engine', 'right-engine', 'cockpit'])

function selectZone(zone) {
  if (selectedZones.has(zone)) {
    selectedZones.delete(zone)
  } else {
    selectedZones.add(zone)
  }
  const zones = [...selectedZones]
  const hasCorrect = zones.some(z => correctZones.has(z))
  const hasWrong   = zones.some(z => !correctZones.has(z))
  const verdictType = hasCorrect && !hasWrong ? 'correct'
                    : hasCorrect && hasWrong  ? 'partial'
                    : zones.length > 0        ? 'wrong'
                    : 'none'
  emit('select', { zones, verdictType })
}

function crackPath(x, y) {
  return [
    `M${x-3} ${y-2} L${x-6} ${y-5}`,
    `M${x+3} ${y-2} L${x+6} ${y-5}`,
    `M${x}   ${y+3} L${x}   ${y+7}`,
    `M${x-2} ${y+2} L${x-5} ${y+5}`,
  ].join(' ')
}

// 返航弹孔（翼面中段 + 机身，整体上移35px）
const bulletHoles = [
  // 左翼外段
  { x:68,  y:167, delay:120, rust:'#7a3c0f' },
  { x:82,  y:172, delay:200 },
  { x:96,  y:169, delay:300 },
  { x:74,  y:177, delay:420 },
  // 左翼内段
  { x:162, y:163, delay:500 },
  { x:174, y:167, delay:580 },
  { x:168, y:173, delay:660 },
  { x:155, y:170, delay:740, rust:'#7a3c0f' },
  // 右翼内段
  { x:238, y:163, delay:160 },
  { x:250, y:167, delay:250 },
  { x:244, y:173, delay:340 },
  // 右翼外段
  { x:318, y:167, delay:450 },
  { x:330, y:172, delay:530 },
  { x:342, y:168, delay:610 },
  { x:324, y:177, delay:690, rust:'#7a3c0f' },
  // 机身（中段密集）
  { x:200, y:130, delay:820 },
  { x:208, y:145, delay:880 },
  { x:198, y:161, delay:940 },
  { x:206, y:177, delay:1000 },
  { x:200, y:193, delay:1060 },
  { x:207, y:209, delay:1120 },
  { x:199, y:225, delay:1180 },
]

// 未返航弹孔（发动机 + 驾驶舱，整体上移35px）
const ghostHoles = [
  // 左外发动机
  { x:86,  y:157 }, { x:94,  y:163 }, { x:90,  y:153 },
  // 左内发动机
  { x:142, y:153 }, { x:150, y:159 }, { x:146, y:149 },
  // 右内发动机
  { x:258, y:153 }, { x:266, y:159 }, { x:262, y:149 },
  // 右外发动机
  { x:310, y:157 }, { x:318, y:163 }, { x:314, y:153 },
  // 驾驶舱
  { x:200, y:71  }, { x:208, y:79  }, { x:202, y:65  },
  { x:196, y:83  }, { x:210, y:85  },
]
</script>

<style scoped>
.plane-map-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.plane-svg {
  width: 100%;
  max-width: 440px;
  border-radius: 3px;
  box-shadow:
    0 4px 24px rgba(0,0,0,0.45),
    0 0 0 1px rgba(180,150,80,0.25);
}

/* ── 弹孔打入动画 ── */
.hole-punch {
  opacity: 0;
  transform-origin: center;
  animation: punchIn 0.18s ease-out forwards;
}
@keyframes punchIn {
  0%   { opacity: 0; transform: scale(2.2); }
  55%  { opacity: 1; transform: scale(0.85); }
  80%  { transform: scale(1.08); }
  100% { opacity: 1; transform: scale(1); }
}

/* ── 幽灵弹孔渗入动画 ── */
.ghost-bleed {
  opacity: 0;
  animation: bleedIn 0.6s ease-out forwards;
}
@keyframes bleedIn {
  0%   { opacity: 0; transform: scale(0.4); }
  60%  { opacity: 0.9; transform: scale(1.15); }
  100% { opacity: 1;   transform: scale(1); }
}

/* ── 交互区域 ── */
.zone {
  cursor: pointer;
  transition: fill 0.14s;
}
.zone:hover { fill: rgba(74, 96, 128, 0.14); }
.zone.selected {
  fill: rgba(74, 96, 128, 0.22);
  stroke: #4a6080;
  stroke-width: 1.2;
  stroke-dasharray: 4 3;
}

/* ── 选区标签 ── */
.zone-label {
  margin-top: 12px;
  font-family: var(--font-mono);
  font-size: 0.88rem;
  color: var(--amber);
  letter-spacing: 0.06em;
}
.zone-label strong { color: var(--amber-bright); }
</style>
