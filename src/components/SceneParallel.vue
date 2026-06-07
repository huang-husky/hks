<template>
  <div class="scene">
    <div style="margin-bottom: 20px">
      <div class="stamp" style="margin-bottom: 12px">平行宇宙</div>
      <h2>时空回溯法庭</h2>
    </div>

    <p style="margin-bottom: 24px">
      让我们看看，<strong>不同装甲方案</strong>会带来什么样的不同结局。
    </p>

    <!-- 决策历史时间线 -->
    <div class="timeline" v-if="decisionHistory.length > 0">
      <div class="timeline-label">你的决策历史：</div>
      <div class="timeline-items">
        <div v-for="(decision, index) in decisionHistory" :key="index" class="timeline-item">
          <div class="timeline-step">#{{ index + 1 }}</div>
          <div class="timeline-content">
            <div class="timeline-title">
              选择了：
              <span v-for="zone in decision.choice" :key="zone" class="tag">{{ zoneNames[zone] }}</span>
            </div>
            <div class="timeline-meta">
              <span class="timeline-type" :class="decision.type">{{ typeNames[decision.type] }}</span>
              <span class="timeline-survival">
                预计生存率：{{ survivalRate(decision.choice) }}%
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <hr class="divider" />

    <!-- 平行宇宙对比 -->
    <div class="parallel-grid">
      <!-- 你的选择 -->
      <div class="parallel-card your-choice">
        <div class="parallel-header">
          <div class="parallel-title">你的选择</div>
          <div class="parallel-badge your-choice">现在</div>
        </div>
        
        <PlaneMap
          :interactive="false"
          :show-selected-zones="true"
          :selected-zones="latestChoice"
        />

        <div class="parallel-stats">
          <div class="stat-value your-choice">{{ survivalRate(latestChoice) }}%</div>
          <div class="stat-label">预计生存率</div>
        </div>

        <div class="parallel-detail">
          <div class="detail-label">你的选择：</div>
          <div class="detail-tags">
            <span v-for="zone in latestChoice" :key="zone" class="tag">{{ zoneNames[zone] }}</span>
          </div>
        </div>
      </div>

      <!-- 最优解 -->
      <div class="parallel-card optimal">
        <div class="parallel-header">
          <div class="parallel-title">最优方案</div>
          <div class="parallel-badge optimal">真相</div>
        </div>
        
        <PlaneMap
          :interactive="false"
          :show-selected-zones="true"
          :selected-zones="optimalChoice"
        />

        <div class="parallel-stats">
          <div class="stat-value optimal">92%</div>
          <div class="stat-label">预计生存率</div>
        </div>

        <div class="parallel-detail">
          <div class="detail-label">最优选择：</div>
          <div class="detail-tags">
            <span v-for="zone in optimalChoice" :key="zone" class="tag green">{{ zoneNames[zone] }}</span>
          </div>
        </div>
      </div>
    </div>

    <hr class="divider" />

    <!-- 分析面板 -->
    <div class="analysis-panel">
      <h3>决策分析</h3>
      
      <div class="analysis-items">
        <div class="analysis-item" v-if="hasWrongZone">
          <span class="analysis-icon">❌</span>
          <span class="analysis-text">
            <strong>错误：</strong>你选择了不需要加固的位置。这些区域弹孔多，
            说明<strong>中弹后仍能返航</strong>。
          </span>
        </div>
        
        <div class="analysis-item" v-if="hasMissedZone">
          <span class="analysis-icon">⚠️</span>
          <span class="analysis-text">
            <strong>遗漏：</strong>你没有加固真正致命的位置。这些区域弹孔少，
            说明<strong>中弹后无法返航</strong>。
          </span>
        </div>
        
        <div class="analysis-item success" v-if="!hasWrongZone && !hasMissedZone">
          <span class="analysis-icon">✅</span>
          <span class="analysis-text">
            <strong>完美！</strong>你找到了真正需要加固的致命区域！
          </span>
        </div>
      </div>
    </div>

    <hr class="divider" />

    <div style="display:flex; gap:12px; justify-content:center; margin-top:16px">
      <button class="btn" @click="$emit('retry')">
        ← 再次回到过去
      </button>
      <button class="btn" @click="$emit('next')">
        进入知识殿堂 →
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed, inject } from 'vue'
import PlaneMap from './PlaneMap.vue'

const props = defineProps({
  latestChoice: { type: Set, required: true },
  decisionHistory: { type: Array, default: () => [] },
})
const emit = defineEmits(['retry', 'next'])

const verdictTypeRef = inject('verdictType')

const optimalChoice = computed(() => new Set(['left-engine', 'right-engine', 'cockpit']))
const zoneNames = {
  'left-wing': '左机翼',
  'right-wing': '右机翼',
  'fuselage': '机身',
  'left-engine': '左发动机',
  'right-engine': '右发动机',
  'cockpit': '驾驶舱',
}
const typeNames = {
  'correct': '判断正确',
  'partial': '部分正确',
  'wrong': '判断有误',
}

const hasWrongZone = computed(() => {
  const wrongZones = ['left-wing', 'right-wing', 'fuselage']
  return wrongZones.some(z => props.latestChoice.has(z))
})
const hasMissedZone = computed(() => {
  const needZones = ['left-engine', 'right-engine', 'cockpit']
  return !needZones.every(z => props.latestChoice.has(z))
})

function survivalRate(choice) {
  const needZones = ['left-engine', 'right-engine', 'cockpit']
  const wrongZones = ['left-wing', 'right-wing', 'fuselage']
  
  let score = 30
  
  needZones.forEach(z => {
    if (choice.has(z)) score += 20
  })
  
  wrongZones.forEach(z => {
    if (choice.has(z)) score -= 5
  })
  
  return Math.max(25, Math.min(92, score))
}
</script>

<style scoped>
.parallel-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin: 24px 0;
}

@media (max-width: 700px) {
  .parallel-grid { grid-template-columns: 1fr; }
}

.parallel-card {
  border: 2px solid var(--border);
  border-radius: 6px;
  padding: 16px;
  background: var(--bg-card);
  transition: all 0.25s ease;
}

.parallel-card:hover {
  box-shadow: 0 4px 16px rgba(0,0,0,0.2);
  transform: translateY(-2px);
}

.parallel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border);
}

.parallel-title {
  font-weight: bold;
  font-size: 1.05rem;
}

.parallel-badge {
  font-size: 0.75rem;
  padding: 4px 12px;
  border-radius: 20px;
  font-family: var(--font-mono);
  letter-spacing: 0.08em;
}
.parallel-badge.your-choice {
  background: rgba(200,168,75,0.12);
  color: var(--amber);
  border: 1px solid var(--amber-dim);
}
.parallel-badge.optimal {
  background: rgba(42,106,42,0.12);
  color: #7aca7a;
  border: 1px solid #2a6a2a;
}
[data-theme="light"] .parallel-badge.optimal {
  background: rgba(42,106,42,0.1);
  color: #2a6a2a;
  border-color: #4a8a4a;
}

.parallel-stats {
  text-align: center;
  margin-top: 14px;
  padding-top: 12px;
  border-top: 1px dashed var(--border);
}

.stat-value {
  font-size: 2.2rem;
  font-family: var(--font-display);
  letter-spacing: 0.08em;
}
.stat-value.your-choice { color: var(--amber); }
.stat-value.optimal { color: #7aca7a; }
[data-theme="light"] .stat-value.optimal { color: #2a6a2a; }

.stat-label {
  font-size: 0.8rem;
  color: var(--text-dim);
  font-family: var(--font-mono);
  letter-spacing: 0.08em;
  margin-top: 4px;
}

.parallel-detail {
  margin-top: 10px;
}

.detail-label {
  font-size: 0.8rem;
  color: var(--text-dim);
  margin-bottom: 6px;
}

.detail-tags {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.tag {
  font-size: 0.75rem;
  padding: 4px 10px;
  border: 1px solid var(--amber-dim);
  color: var(--amber);
  border-radius: 4px;
  font-family: var(--font-mono);
}
.tag.green {
  border-color: #2a6a2a;
  color: #7aca7a;
}
[data-theme="light"] .tag.green {
  border-color: #4a8a4a;
  color: #2a6a2a;
}

/* 时间线 */
.timeline {
  margin-bottom: 20px;
}

.timeline-label {
  font-size: 0.85rem;
  color: var(--text-dim);
  margin-bottom: 8px;
  font-family: var(--font-mono);
  letter-spacing: 0.08em;
}

.timeline-items {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.timeline-item {
  display: flex;
  gap: 10px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 4px;
  padding: 10px 14px;
  align-items: center;
}

.timeline-step {
  font-size: 0.85rem;
  color: var(--text-dim);
  font-family: var(--font-display);
  padding-right: 8px;
  border-right: 1px solid var(--border);
}

.timeline-content {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.timeline-title {
  font-size: 0.9rem;
}

.timeline-meta {
  display: flex;
  gap: 8px;
  align-items: center;
}

.timeline-type {
  font-size: 0.75rem;
  font-family: var(--font-mono);
  letter-spacing: 0.08em;
  padding: 2px 8px;
  border-radius: 3px;
}
.timeline-type.correct { background: rgba(42,106,42,0.15);  color: #7aca7a; }
.timeline-type.partial { background: rgba(200,168,75,0.15); color: var(--amber); }
.timeline-type.wrong   { background: rgba(192,57,43,0.15);  color: #e07060; }
[data-theme="light"] .timeline-type.correct { background: rgba(42,106,42,0.12);  color: #2a6a2a; }
[data-theme="light"] .timeline-type.wrong   { background: rgba(139,26,26,0.10);  color: #8b1a1a; }

.timeline-survival {
  font-size: 0.8rem;
  color: var(--text-dim);
}

/* 分析面板 */
.analysis-panel {
  padding: 16px 20px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 4px;
}

.analysis-panel h3 {
  color: var(--amber-bright);
  margin-bottom: 12px;
  font-size: 1rem;
}

.analysis-items {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.analysis-item {
  display: flex;
  gap: 10px;
  align-items: flex-start;
  padding: 8px 0;
  border-bottom: 1px dashed var(--border);
}
.analysis-item:last-child { border-bottom: none; }

.analysis-item.success {
  padding: 10px 12px;
  background: rgba(42,106,42,0.12);
  border-radius: 4px;
}
[data-theme="light"] .analysis-item.success {
  background: rgba(42,106,42,0.08);
  border-left: 2px solid #4a8a4a;
}

.analysis-icon {
  font-size: 1.2rem;
  margin-top: 1px;
}

.analysis-text {
  font-size: 0.9rem;
  line-height: 1.6;
}
</style>
