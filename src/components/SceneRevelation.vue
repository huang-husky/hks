<template>
  <div class="scene">
    <!-- Result banner -->
    <div class="result-banner fade-in" :class="bannerClass">
      <span class="stamp">{{ bannerStamp }}</span>
      <h2 style="margin-top:12px">{{ bannerTitle }}</h2>
    </div>

    <!-- Consequence text by verdict type -->
    <div v-if="vType === 'wrong'" class="card fade-in" style="margin:20px 0; border-color:var(--red)">
      <p style="color:#e07060">
        根据你的建议，工程师在弹孔最多的机翼和机身加装了额外装甲。<br>
        接下来三个月，B-17 的损失率没有下降——反而略有上升。<br>
        飞行员们继续在欧洲的天空上消失。
      </p>
    </div>
    <div v-else-if="vType === 'partial'" class="card fade-in" style="margin:20px 0; border-color:var(--amber-dim)">
      <p style="color:var(--amber)">
        你的判断触及了一部分真相——你注意到了那些弹孔稀少的区域。<br>
        但同时你也选择了弹孔密集的位置，说明直觉还在干扰你的推理。<br>
        让我们来看看，你差一步没想到的是什么。
      </p>
    </div>

    <!-- Revelation: the ghost planes -->
    <div class="revelation-section fade-in" style="margin:28px 0">
      <h3 style="margin-bottom:16px; color:var(--amber-bright)">— 真相 —</h3>

      <div class="two-col">
        <div>
          <p style="margin-bottom:12px; font-size:0.82rem; color:var(--amber-dim); letter-spacing:0.08em">
            你看到的数据：返航飞机的弹孔
          </p>
          <PlaneMap :show-bullet-holes="true" :interactive="false" />
        </div>
        <div>
          <p style="margin-bottom:12px; font-size:0.82rem; color:#c0392b; letter-spacing:0.08em">
            你没看到的数据：未返航飞机的致命命中区域
          </p>
          <PlaneMap :show-bullet-holes="false" :show-ghost="true" :interactive="false" />
        </div>
      </div>

      <div class="insight-box fade-in">
        <p>
          翼面和机身弹孔密集，<strong>恰恰说明这些位置中弹后飞机还能回来</strong>。<br>
          发动机和驾驶舱在返航数据中几乎看不到弹孔——<br>
          <strong>因为被击中发动机的飞机，根本没有机会返航。</strong>
        </p>
      </div>
    </div>

    <!-- Wald card -->
    <div class="wald-card card fade-in">
      <div style="display:flex; align-items:flex-start; gap:16px">
        <span style="font-size:2rem">📐</span>
        <div>
          <div style="font-size:0.78rem; color:var(--amber-dim); letter-spacing:0.1em; margin-bottom:4px">历史档案</div>
          <p>
            这个洞见来自真实存在的人：<strong style="color:var(--amber-bright)">Abraham Wald</strong>，
            匈牙利裔数学家，1943年哥伦比亚大学统计研究组。<br><br>
            当所有工程师都盯着弹孔最多的地方时，Wald 问了一个不同的问题：<br>
            <em>「哪些飞机没有出现在我的数据里？」</em><br><br>
            他的分析拯救了成千上万名飞行员的生命。
          </p>
        </div>
      </div>
    </div>

    <!-- Academic connection -->
    <div class="academic-box card fade-in">
      <h3 style="margin-bottom:14px">这在课本里叫什么？</h3>
      <div class="concept-row">
        <!-- 幸存者偏差 -->
        <div 
          class="concept concept-clickable"
          @click="openConcept('survivorship')"
        >
          <div class="concept-name">幸存者偏差</div>
          <div class="concept-desc">只观察「幸存下来」的样本，忽略了没有进入视野的数据</div>
          <div class="concept-hint">点击查看详情 →</div>
        </div>
        
        <div class="arrow">→</div>
        
        <!-- 选择性偏误 -->
        <div 
          class="concept concept-clickable"
          @click="openConcept('selection')"
        >
          <div class="concept-name">选择性偏误</div>
          <div class="concept-desc">样本不是随机抽取的，而是由结果决定的</div>
          <div class="concept-hint">点击查看详情 →</div>
        </div>
        
        <div class="arrow">→</div>
        
        <!-- 遗漏变量偏误 -->
        <div 
          class="concept concept-clickable"
          @click="openConcept('omitted')"
        >
          <div class="concept-name">遗漏变量偏误</div>
          <div class="concept-desc">计量经济学的核心问题：你的回归遗漏了关键变量</div>
          <div class="concept-hint">点击查看详情 →</div>
        </div>
      </div>
      <p style="margin-top:16px; font-size:0.82rem; color:var(--text-dim); text-align:center">
        同一个问题，不同的名字。教材把它写成公式，Wald 用飞机和生命解释了它。
      </p>
    </div>

    <!-- Closing quote -->
    <div class="closing fade-in">
      <blockquote>
        「我们看到的，告诉我们飞机能承受什么。<br>
        我们看不到的，才告诉我们飞机会在哪里折断。」
      </blockquote>
      <cite>— Abraham Wald，1943</cite>
    </div>

    <div style="display:flex; justify-content:center; margin-top:36px">
      <button class="btn" @click="restart">重新开始 ↺</button>
    </div>

    <!-- 小黑板弹窗 -->
    <ConceptModal 
      :show="showModal" 
      :concept="selectedConcept"
      @close="showModal = false" 
    />
  </div>
</template>

<script setup>
import { inject, computed, ref } from 'vue'
import PlaneMap from './PlaneMap.vue'
import ConceptModal from './ConceptModal.vue'
import { concepts } from '../data/concepts.js'

const verdictCorrect = inject('verdictCorrect')
const verdictTypeRef  = inject('verdictType')

const showModal = ref(false)
const selectedConcept = ref(null)

const bannerClass = computed(() => ({
  correct: verdictTypeRef?.value === 'correct',
  partial: verdictTypeRef?.value === 'partial',
  wrong:   verdictTypeRef?.value === 'wrong',
}))
const bannerStamp = computed(() =>
  verdictTypeRef?.value === 'correct' ? '判断正确' : verdictTypeRef?.value === 'partial' ? '部分正确' : '判断有误'
)
const bannerTitle = computed(() =>
  verdictTypeRef?.value === 'correct' ? '你看穿了数据的幻象。'
: verdictTypeRef?.value === 'partial' ? '你触及了真相，但直觉还在干扰你。'
: '你的装甲，加错了地方。'
)

function restart() {
  window.location.reload()
}

function openConcept(conceptId) {
  selectedConcept.value = concepts.find(c => c.id === conceptId)
  showModal.value = true
}
</script>

<style scoped>
.result-banner {
  text-align: center;
  padding: 24px;
  margin-bottom: 8px;
  border: 1px solid;
}
.result-banner.correct { 
  border-color: #2a6a2a;       
  background: #0a180a; 
}
.result-banner.partial { 
  border-color: var(--amber-dim); 
  background: #140e00; 
}
.result-banner.wrong   { 
  border-color: var(--red);    
  background: #180a0a; 
}

.two-col {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  margin-bottom: 20px;
  text-align: center;
}
@media (max-width: 600px) {
  .two-col { grid-template-columns: 1fr; }
}

.insight-box {
  background: #0a180a;
  border: 1px solid #2a4a2a;
  padding: 18px 22px;
  text-align: center;
}
.insight-box p { font-size: 0.9rem; color: var(--text); line-height: 1.75; }
.insight-box strong { color: var(--amber-bright); }

.wald-card {
  margin: 24px 0;
  border-color: var(--amber-dim);
}
.wald-card p { font-size: 0.88rem; line-height: 1.75; }
.wald-card strong { color: var(--amber-bright); }
.wald-card em { color: var(--amber); font-style: italic; }

.academic-box { margin: 24px 0; }
.concept-row {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}
.concept {
  flex: 1;
  min-width: 140px;
  padding: 12px;
  background: var(--bg);
  border: 1px solid var(--border);
  text-align: center;
}
.concept-name {
  font-size: 0.88rem;
  color: var(--amber-bright);
  letter-spacing: 0.06em;
  margin-bottom: 6px;
}
.concept-desc {
  font-family: var(--font-mono);
  font-size: 0.72rem;
  color: var(--text-dim);
  line-height: 1.5;
}

/* ── 可点击概念卡片样式 ── */
.concept-clickable {
  cursor: pointer;
  transition: 
    all 0.25s ease,
    transform 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

.concept-clickable::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    135deg,
    rgba(200,168,75,0.03) 0%,
    transparent 50%
  );
  opacity: 0;
  transition: opacity 0.3s ease;
}

.concept-clickable:hover {
  border-color: var(--amber);
  background: rgba(200,168,75,0.05);
  transform: translateY(-3px);
  box-shadow: 
    0 6px 16px rgba(0,0,0,0.2),
    0 0 12px rgba(200,168,75,0.1);
}

.concept-clickable:hover::before {
  opacity: 1;
}

.concept-clickable:hover .concept-name {
  color: var(--amber-bright);
  text-shadow: 0 0 8px rgba(240,204,106,0.4);
}

.concept-clickable:active {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}

.concept-hint {
  font-family: var(--font-mono);
  font-size: 0.65rem;
  color: var(--amber-dim);
  margin-top: 8px;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.concept-clickable:hover .concept-hint {
  opacity: 1;
}

.arrow {
  color: var(--amber-dim);
  font-size: 1.2rem;
  flex-shrink: 0;
}

.closing {
  text-align: center;
  margin-top: 40px;
  padding: 24px;
  border-top: 1px solid var(--border);
}
blockquote {
  font-family: var(--font-hand);
  font-size: 1.45rem;
  color: var(--amber);
  line-height: 1.9;
  font-style: normal;
  margin-bottom: 12px;
  text-shadow: var(--glow-h2);
}
cite {
  font-family: var(--font-mono);
  font-size: 0.78rem;
  color: var(--text-dim);
  letter-spacing: 0.1em;
}

/* ── 浅色模式适配 ── */
[data-theme="light"] .result-banner.correct { 
  border-color: #a0c8a0; 
  background: #e8f0e8; 
}
[data-theme="light"] .result-banner.partial { 
  border-color: #c8a870; 
  background: #f0e8d8; 
}
[data-theme="light"] .result-banner.wrong { 
  border-color: #c8a0a0; 
  background: #f8e8e8; 
}

[data-theme="light"] .insight-box {
  background: #e8f0e8;
  border-color: #a0c8a0;
}

[data-theme="light"] blockquote {
  text-shadow: none;
}

[data-theme="light"] .concept {
  background: var(--bg-card);
}

[data-theme="light"] .concept-name {
  text-shadow: none;
}

[data-theme="light"] .concept-clickable:hover {
  box-shadow: 0 4px 12px rgba(90,54,8,0.15);
}

[data-theme="light"] .concept-clickable:hover .concept-name {
  text-shadow: none;
}

/* ── 深色模式保持深色 ── */
:not([data-theme="light"]) .result-banner.correct { 
  background: #0a180a; 
}
:not([data-theme="light"]) .result-banner.partial { 
  background: #140e00; 
}
:not([data-theme="light"]) .result-banner.wrong { 
  background: #180a0a; 
}
:not([data-theme="light"]) .insight-box {
  background: #0a180a;
}
</style>
