<template>
  <div class="chat-wrap" :style="{ '--witness-color': witness.color }">
    <div class="chat-header">
      <div class="witness-avatar-badge">
        <span class="witness-avatar">{{ witness.avatar }}</span>
      </div>
      <div>
        <div class="witness-name">{{ witness.name }}</div>
        <div class="witness-title">{{ witness.title }}</div>
      </div>
      <button class="btn btn-sm btn-clear" @click="clearMessages" title="清空对话记录">清空</button>
    </div>

    <div class="chat-log" ref="logEl">
      <div v-if="messages.length === 0" class="chat-empty">
        开口问吧。他在等着。
      </div>
      <div
        v-for="(msg, i) in messages"
        :key="i"
        class="chat-msg"
        :class="msg.role"
      >
        <span class="msg-role">{{ msg.role === 'user' ? '你' : witness.name }}</span>
        <p>{{ msg.content }}</p>
      </div>
      <div v-if="loading" class="chat-msg assistant">
        <span class="msg-role">{{ witness.name }}</span>
        <p class="thinking">···</p>
      </div>
    </div>

    <div class="chat-input-row">
      <input
        v-model="inputText"
        ref="inputEl"
        class="chat-input"
        :placeholder="'问 ' + witness.name + ' 任何问题...'"
        @keydown.enter="sendMessage"
        :disabled="loading"
      />
      <button class="btn btn-sm btn-send" @click="sendMessage" :disabled="loading || !inputText.trim()">
        发送
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'
import { askWitness } from '../api/deepseek.js'

const props = defineProps({
  witness: { type: Object, required: true },
  autoFocus: { type: Boolean, default: false },
})
const messages = ref([])
const inputText = ref('')
const loading = ref(false)
const logEl = ref(null)
const inputEl = ref(null)

onMounted(() => {
  if (props.autoFocus && inputEl.value) {
    inputEl.value.focus()
  }
})

async function sendMessage() {
  const text = inputText.value.trim()
  if (!text || loading.value) return

  messages.value.push({ role: 'user', content: text })
  inputText.value = ''
  loading.value = true
  await scrollToBottom()

  try {
    const apiMessages = messages.value.map(m => ({ role: m.role, content: m.content }))
    const reply = await askWitness(apiMessages, props.witness.systemPrompt)
    messages.value.push({ role: 'assistant', content: reply })
  } catch (e) {
    messages.value.push({ role: 'assistant', content: '[ 通讯中断，请重试 ]' })
  } finally {
    loading.value = false
    await scrollToBottom()
  }
}

async function scrollToBottom() {
  await nextTick()
  if (logEl.value) logEl.value.scrollTop = logEl.value.scrollHeight
}

function clearMessages() {
  messages.value = []
}
</script>

<style scoped>
.chat-wrap {
  display: flex;
  flex-direction: column;
  height: 320px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 6px;
  overflow: hidden;
  transition: box-shadow 0.4s ease-out, transform 0.4s ease-out;
}
.chat-wrap:hover {
  box-shadow: 0 0 0 1.5px color-mix(in srgb, var(--witness-color) 25%, transparent),
              0 3px 8px rgba(0,0,0,0.1);
  transform: translateY(-1.5px);
}
.chat-header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 16px;
  border-bottom: 1px solid var(--border);
  background: color-mix(in srgb, var(--witness-color) 10%, var(--bg-card));
}
.witness-avatar-badge {
  width: 46px;
  height: 46px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid var(--witness-color);
  background: color-mix(in srgb, var(--witness-color) 15%, var(--bg-card));
}
.witness-avatar {
  font-size: 1.6rem;
  filter: drop-shadow(0 0 4px var(--witness-color));
}
.witness-name {
  font-family: var(--font-elegant);
  font-size: 0.98rem;
  color: var(--witness-color);
  letter-spacing: 0.06em;
}
.witness-title {
  font-family: var(--font-mono);
  font-size: 0.7rem;
  color: var(--text-dim);
}
.chat-log {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.chat-empty {
  font-family: var(--font-mono);
  font-size: 0.82rem;
  color: var(--text-dim);
  text-align: center;
  margin-top: 60px;
  letter-spacing: 0.1em;
}
.chat-msg { 
  display: flex; 
  flex-direction: column; 
  gap: 3px; 
  animation: slideInSoft 0.3s ease-out;
}
@keyframes slideInSoft {
  from { opacity: 0; transform: translateY(6px); }
  to { opacity: 1; transform: translateY(0); }
}
.chat-msg.user { align-items: flex-end; }
.chat-msg.assistant { align-items: flex-start; }
.msg-role {
  font-size: 0.7rem;
  color: var(--text-dim);
  letter-spacing: 0.1em;
  text-transform: uppercase;
  font-family: var(--font-mono);
  margin-bottom: 2px;
}
.chat-msg p {
  max-width: 82%;
  padding: 8px 12px;
  font-size: 0.88rem;
  line-height: 1.6;
}
.chat-msg.user p {
  background: var(--bg-msg-user);
  border: 1px solid var(--border-msg-user);
  color: var(--color-msg-user);
}
.chat-msg.assistant p {
  background: color-mix(in srgb, var(--witness-color) 8%, var(--bg-msg-assist));
  border: 1px solid color-mix(in srgb, var(--witness-color) 35%, var(--border-msg-assist));
  color: var(--color-msg-assist);
}
.thinking {
  letter-spacing: 0.3em;
  animation: pulse 1.8s ease-in-out infinite;
}
@keyframes pulse { 0%,100%{opacity:0.4} 50%{opacity:0.9} }
.chat-input-row {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  border-top: 1px solid var(--border);
}
.chat-input {
  flex: 1;
  background: var(--bg-input);
  border: 1px solid var(--border);
  color: var(--amber);
  font-family: var(--font-mono);
  font-size: 0.85rem;
  padding: 8px 12px;
  outline: none;
}
.chat-input:focus { border-color: var(--witness-color); }
.chat-input::placeholder { color: var(--text-dim); }
.btn-clear {
  background: color-mix(in srgb, var(--witness-color) 10%, transparent);
  border-color: color-mix(in srgb, var(--witness-color) 30%, var(--border));
}
.btn-send {
  background: color-mix(in srgb, var(--witness-color) 12%, var(--bg-card));
  border-color: color-mix(in srgb, var(--witness-color) 45%, var(--border));
}

/* ── 浅色模式适配 ── */
[data-theme="light"] .witness-name {
  text-shadow: none;
}
</style>
