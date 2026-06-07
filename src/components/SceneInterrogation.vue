<template>
  <div class="scene">
    <div style="margin-bottom:8px">
      <div class="stamp" style="margin-bottom:12px">第二幕</div>
      <h2>审讯证人</h2>
    </div>
    <p style="margin-bottom:28px">
      在做出判断之前，你可以审讯以下三位证人。<br>
      他们每个人知道的事情都不一样——问题问得越精准，线索越有价值。
    </p>

    <!-- 三个窗口始终并排 -->
    <div class="chat-grid">
      <WitnessChat v-for="(w, index) in witnesses" :key="w.id" :witness="w" :auto-focus="index === 0" />
    </div>

    <hr class="divider" />

    <div class="submit-btn-wrapper">
      <button class="btn" @click="$emit('next')">
        提交我的判断 →
      </button>
    </div>
  </div>
</template>

<script setup>
import { witnesses } from '../data/witnesses.js'
import WitnessChat from './WitnessChat.vue'

defineEmits(['next'])
</script>

<style scoped>
.chat-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
  margin-bottom: 32px;
}

.submit-btn-wrapper {
  padding-top: 16px;
  position: sticky;
  bottom: 0;
  background: var(--bg);
  padding-bottom: 16px;
  z-index: 10;
}

@media (max-width: 900px) {
  .chat-grid { 
    grid-template-columns: 1fr; 
  }
  
  .chat-wrap {
    height: 280px;
  }
}
</style>
