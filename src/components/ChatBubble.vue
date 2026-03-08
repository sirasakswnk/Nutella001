<template>
  <div class="bubble-row" :class="msg.role">
    <div class="avatar" :class="msg.role">
      {{ msg.role === 'ai' ? '🩺' : 'คุณ' }}
    </div>
    <div class="bubble-wrap">
      <div class="bubble" :class="msg.role" v-html="formattedContent"></div>
      <div class="bubble-time">{{ msg.time }}</div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  msg: { type: Object, required: true }
})

const formattedContent = computed(() =>
  props.msg.content
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/\n/g, '<br/>')
)
</script>

<style scoped>
.bubble-row {
  display: flex;
  gap: 10px;
  align-items: flex-end;
  animation: fadeUp .25s ease both;
}
.bubble-row.user { flex-direction: row-reverse; }

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.avatar {
  width: 34px; height: 34px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 16px;
  flex-shrink: 0;
}
.avatar.ai   { background: var(--green-pale); border: 1px solid var(--border); }
.avatar.user { background: var(--green-deep); color: #fff; font-size: 11px; font-weight: 600; }

.bubble-wrap { display: flex; flex-direction: column; max-width: 72%; }
.bubble-row.user .bubble-wrap { align-items: flex-end; }

.bubble {
  padding: 12px 16px;
  border-radius: 16px;
  font-size: 14.5px;
  line-height: 1.75;
  box-shadow: 0 2px 12px rgba(26,74,58,.07);
  word-break: break-word;
}
.bubble.ai {
  background: #fff;
  border: 1px solid var(--border);
  border-bottom-left-radius: 4px;
  color: var(--ink);
}
.bubble.user {
  background: var(--green-deep);
  color: #fff;
  border-bottom-right-radius: 4px;
}

.bubble-time {
  font-size: 11px;
  color: var(--ink-soft);
  opacity: .45;
  margin-top: 4px;
  padding: 0 4px;
}
</style>