<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="show" class="modal-overlay" @click.self="$emit('close')">
        <div class="blackboard">
          <!-- 黑板顶部装饰 -->
          <div class="blackboard-header">
            <div class="chalk-tray">
              <div class="chalk-piece chalk-yellow"></div>
              <div class="chalk-piece chalk-white"></div>
              <div class="chalk-piece chalk-red"></div>
            </div>
          </div>

          <!-- 黑板内容区域 -->
          <div class="blackboard-content">
            <!-- 标题区域 -->
            <div class="title-section">
              <h2 class="chalk-title">{{ concept?.name }}</h2>
              <p class="chalk-subtitle">{{ concept?.title }}</p>
            </div>

            <!-- 分隔线 -->
            <div class="chalk-line"></div>

            <!-- 描述 -->
            <div class="desc-section">
              <p class="chalk-desc">{{ concept?.description }}</p>
            </div>

            <!-- 主要内容 -->
            <div class="content-section">
              <div 
                v-for="(paragraph, idx) in contentParagraphs" 
                :key="idx"
                class="chalk-paragraph"
              >
                <div v-if="isHeading(paragraph)" class="chalk-heading">
                  {{ paragraph }}
                </div>
                <div v-else-if="isListItem(paragraph)" class="chalk-list-item">
                  <span class="list-bullet">✦</span>
                  <span>{{ paragraph.slice(2) }}</span>
                </div>
                <div v-else-if="isMath(paragraph)" class="chalk-math">
                  {{ paragraph }}
                </div>
                <div v-else class="chalk-text">
                  {{ paragraph }}
                </div>
              </div>
            </div>

            <!-- 例子 -->
            <div class="examples-section">
              <div class="chalk-heading">现实例子：</div>
              <div class="examples-grid">
                <div 
                  v-for="(ex, idx) in concept?.examples" 
                  :key="idx"
                  class="example-tag"
                >
                  {{ ex }}
                </div>
              </div>
            </div>

            <!-- 关键启示 -->
            <div class="takeaway-section">
              <div class="takeaway-box">
                <span class="takeaway-icon">💡</span>
                <span class="takeaway-text">{{ concept?.keyTakeaway }}</span>
              </div>
            </div>
          </div>

          <!-- 黑板底部装饰 -->
          <div class="blackboard-footer">
            <div class="eraser"></div>
          </div>

          <!-- 关闭按钮 -->
          <button class="close-btn" @click="$emit('close')">
            <span class="close-icon">✕</span>
          </button>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  show: Boolean,
  concept: Object
})

defineEmits(['close'])

const contentParagraphs = computed(() => {
  if (!props.concept?.content) return []
  return props.concept.content.split('\n').filter(p => p.trim())
})

function isHeading(text) {
  return text.includes('：') && text.length < 20
}

function isListItem(text) {
  return text.startsWith('- ')
}

function isMath(text) {
  return text.includes('=') || text.includes('β') || text.includes('ε') || text.includes('α') || text.includes('X₁')
}
</script>

<style scoped>
/* ── 弹窗背景 ── */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  padding: 20px;
}

/* ── 小黑板主体 ── */
.blackboard {
  width: 100%;
  max-width: 720px;
  max-height: 85vh;
  background: linear-gradient(
    180deg,
    #1a2a1a 0%,
    #122012 50%,
    #0d180d 100%
  );
  border-radius: 8px;
  box-shadow: 
    0 0 30px rgba(0, 0, 0, 0.5),
    inset 0 0 60px rgba(0, 0, 0, 0.3);
  position: relative;
  overflow: hidden;
}

/* ── 黑板顶部 - 粉笔槽 ── */
.blackboard-header {
  background: #0a120a;
  padding: 8px 0;
  border-bottom: 3px solid #081008;
}

.chalk-tray {
  display: flex;
  justify-content: center;
  gap: 8px;
}

.chalk-piece {
  width: 24px;
  height: 6px;
  border-radius: 3px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
}

.chalk-yellow { background: #f0cc6a; }
.chalk-white { background: #e8e8e8; }
.chalk-red { background: #c0392b; }

/* ── 黑板内容 ── */
.blackboard-content {
  padding: 28px 32px;
  max-height: 65vh;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: #4a6a4a #0d180d;
}

/* ── 标题区域 ── */
.title-section {
  text-align: center;
  margin-bottom: 20px;
}

.chalk-title {
  font-family: 'Special Elite', 'Courier New', monospace;
  font-size: 2rem;
  color: #f0cc6a;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  text-shadow: 
    0 0 8px rgba(240, 204, 106, 0.4),
    2px 2px 0 rgba(0, 0, 0, 0.3);
  margin: 0 0 8px;
}

.chalk-subtitle {
  font-family: 'Courier New', monospace;
  font-size: 0.85rem;
  color: #8a9a8a;
  letter-spacing: 0.1em;
  margin: 0;
}

/* ── 粉笔线分隔 ── */
.chalk-line {
  width: 100%;
  height: 2px;
  background: repeating-linear-gradient(
    90deg,
    #f0cc6a 0px,
    #f0cc6a 8px,
    transparent 8px,
    transparent 12px
  );
  margin: 20px 0;
  opacity: 0.6;
}

/* ── 描述 ── */
.desc-section {
  text-align: center;
  margin-bottom: 24px;
}

.chalk-desc {
  font-family: 'Courier New', monospace;
  font-size: 0.95rem;
  color: #c8d8c8;
  line-height: 1.7;
  margin: 0;
  font-style: italic;
}

/* ── 主要内容 ── */
.content-section {
  margin-bottom: 24px;
}

.chalk-paragraph {
  margin-bottom: 14px;
}

.chalk-heading {
  font-family: 'Special Elite', 'Courier New', monospace;
  font-size: 1.1rem;
  color: #f0cc6a;
  letter-spacing: 0.1em;
  margin-bottom: 8px;
  padding-left: 4px;
}

.chalk-text {
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  color: #b8c8b8;
  line-height: 1.8;
  text-align: justify;
}

.chalk-list-item {
  font-family: 'Courier New', monospace;
  font-size: 0.88rem;
  color: #a8b8a8;
  line-height: 1.6;
  display: flex;
  gap: 8px;
  padding-left: 8px;
}

.list-bullet {
  color: #f0cc6a;
  font-size: 0.7rem;
}

.chalk-math {
  font-family: 'Courier New', monospace;
  font-size: 0.95rem;
  color: #d8e8d8;
  background: rgba(240, 204, 106, 0.08);
  padding: 8px 12px;
  border-radius: 4px;
  margin: 4px 0;
  text-align: center;
}

/* ── 例子区域 ── */
.examples-section {
  margin-bottom: 24px;
}

.examples-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 10px;
}

.example-tag {
  font-family: 'Courier New', monospace;
  font-size: 0.78rem;
  color: #c0d0c0;
  background: rgba(240, 204, 106, 0.1);
  border: 1px solid rgba(240, 204, 106, 0.2);
  padding: 6px 12px;
  border-radius: 4px;
}

/* ── 关键启示 ── */
.takeaway-section {
  margin-top: 24px;
}

.takeaway-box {
  background: rgba(240, 204, 106, 0.1);
  border-left: 3px solid #f0cc6a;
  padding: 16px 20px;
  display: flex;
  gap: 12px;
  align-items: flex-start;
}

.takeaway-icon {
  font-size: 1.3rem;
}

.takeaway-text {
  font-family: 'Courier New', monospace;
  font-size: 0.92rem;
  color: #d8e8d8;
  line-height: 1.6;
  font-style: italic;
}

/* ── 黑板底部 - 黑板擦 ── */
.blackboard-footer {
  background: #0a120a;
  padding: 10px 0;
  border-top: 3px solid #081008;
  display: flex;
  justify-content: flex-end;
  padding-right: 20px;
}

.eraser {
  width: 40px;
  height: 20px;
  background: #6a5a4a;
  border-radius: 2px;
  box-shadow: 
    inset 0 1px 2px rgba(255, 255, 255, 0.1),
    inset 0 -1px 2px rgba(0, 0, 0, 0.3);
}

/* ── 关闭按钮 ── */
.close-btn {
  position: absolute;
  top: 12px;
  right: 12px;
  width: 32px;
  height: 32px;
  background: rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(240, 204, 106, 0.3);
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.close-btn:hover {
  background: rgba(192, 57, 43, 0.3);
  border-color: #c0392b;
}

.close-icon {
  color: #f0cc6a;
  font-size: 0.9rem;
}

/* ── 弹窗动画 ── */
.modal-enter-active,
.modal-leave-active {
  transition: all 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .blackboard,
.modal-leave-to .blackboard {
  transform: scale(0.95) translateY(20px);
}

/* ── 滚动条样式 ── */
.blackboard-content::-webkit-scrollbar {
  width: 6px;
}

.blackboard-content::-webkit-scrollbar-track {
  background: #0d180d;
}

.blackboard-content::-webkit-scrollbar-thumb {
  background: #4a6a4a;
  border-radius: 3px;
}

.blackboard-content::-webkit-scrollbar-thumb:hover {
  background: #6a8a6a;
}
</style>