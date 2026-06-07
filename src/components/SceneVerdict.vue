<template>
  <div class="scene">
    <div style="margin-bottom:8px">
      <div class="stamp" style="margin-bottom:12px">第三幕</div>
      <h2>提交判断</h2>
    </div>
    <p style="margin-bottom:28px">
      点击飞机图上你认为<strong>应该加固装甲</strong>的位置，可多选。<br>
      <span style="color:var(--text-dim); font-size:0.88rem">再次点击同一位置可取消选择</span>
    </p>

    <PlaneMap
      :show-bullet-holes="true"
      :interactive="true"
      style="margin-bottom:24px"
      @select="onSelect"
    />

    <!-- 已选区域 + 个性化提示 -->
    <div v-if="selectedZones.length > 0" class="hints-area fade-in">
      <div
        v-for="zone in selectedZones"
        :key="zone"
        class="zone-hint"
        :class="zone"
      >
        <span class="zone-tag">{{ zoneNames[zone] }}</span>
        <p>{{ zoneHints[zone] }}</p>
      </div>
    </div>

    <div style="display:flex; justify-content:center; margin-top:28px">
      <button
        class="btn"
        :class="{ 'btn-red': selectedZones.length > 0 }"
        :disabled="selectedZones.length === 0"
        @click="submitVerdict"
      >
        确认提交判断 →
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import PlaneMap from './PlaneMap.vue'

const emit = defineEmits(['verdict'])

const selectedZones = ref([])
const currentVerdictType = ref('none')

const zoneNames = {
  'left-wing':    '左机翼',
  'right-wing':   '右机翼',
  'fuselage':     '机身',
  'left-engine':  '左侧发动机',
  'right-engine': '右侧发动机',
  'cockpit':      '驾驶舱',
}

const zoneHints = {
  'left-wing':
    '弹孔最密集的区域之一。直觉上这里最需要保护——但请问自己：为什么这里有这么多弹孔，飞机却还能飞回来？',
  'right-wing':
    '与左翼一样，弹孔触目惊心。但「弹孔多」和「最需要加固」真的是同一件事吗？',
  'fuselage':
    '机身贯穿全机，中弹数据可观。从数据上看这似乎合理——但这份数据来自哪些飞机？',
  'left-engine':
    '发动机区域弹孔少得出奇。为什么敌方子弹好像「避开」了发动机？还是另有原因？',
  'right-engine':
    '右侧发动机同样弹孔稀少，几乎和左侧一致。这是巧合，还是一种你还没看透的规律？',
  'cockpit':
    '驾驶舱弹孔接近于零。飞行员们真的那么幸运吗——还是说，这里中弹的飞机根本没有机会回来？',
}

function onSelect({ zones, verdictType }) {
  selectedZones.value = zones
  currentVerdictType.value = verdictType
}

function submitVerdict() {
  emit('verdict', {
    correct: currentVerdictType.value === 'correct',
    verdictType: currentVerdictType.value,
    zones: selectedZones.value,
  })
}
</script>

<style scoped>
.hints-area {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.zone-hint {
  padding: 14px 18px;
  border: 1px solid var(--border);
  background: var(--bg-card);
  border-left-width: 3px;
  transition: border-color 0.2s;
}
/* 错误区域 —— 橙色左边框 */
.zone-hint.left-wing,
.zone-hint.right-wing,
.zone-hint.fuselage {
  border-left-color: var(--amber-dim);
}
/* 正确区域 —— 蓝绿左边框 */
.zone-hint.left-engine,
.zone-hint.right-engine,
.zone-hint.cockpit {
  border-left-color: #4a7a6a;
}
.zone-tag {
  display: inline-block;
  font-family: var(--font-mono);
  font-size: 0.78rem;
  letter-spacing: 0.1em;
  color: var(--amber);
  margin-bottom: 6px;
  text-transform: uppercase;
}
.zone-hint p {
  font-size: 0.96rem;
  line-height: 1.75;
  color: var(--text);
}
</style>
