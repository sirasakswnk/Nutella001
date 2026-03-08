<template>
  <Transition name="modal">
    <div v-if="show" class="overlay" @click.self="$emit('close')">
      <div class="modal">
        <div class="modal-icon">⚙️</div>
        <h2>ตั้งค่า API URL</h2>
        <p>วาง <strong>ngrok URL</strong> ที่ได้จาก Google Colab<br/>
          ตัวอย่าง: <code>https://xxxx.ngrok-free.app</code></p>

        <input
          v-model="inputUrl"
          type="text"
          placeholder="https://xxxx-xx-xxx.ngrok-free.app"
          @keydown.enter="save"
          :class="{ error: !!errorMsg }"
        />
        <p v-if="errorMsg" class="error-msg">{{ errorMsg }}</p>

        <button class="btn-save" :disabled="loading" @click="save">
          <span v-if="loading" class="spinner"></span>
          <span v-else>เชื่อมต่อ</span>
        </button>
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({ show: Boolean, initialUrl: String })
const emit  = defineEmits(['saved', 'close'])

const inputUrl = ref(props.initialUrl || '')
const errorMsg = ref('')
const loading  = ref(false)

watch(() => props.initialUrl, v => { if (v) inputUrl.value = v })

async function save() {
  errorMsg.value = ''
  const url = inputUrl.value.trim().replace(/\/$/, '')

  if (!url.startsWith('http')) {
    errorMsg.value = 'URL ต้องขึ้นต้นด้วย https://'
    return
  }

  loading.value = true
  try {
    const res = await fetch(`${url}/health`, {
      headers: { 'ngrok-skip-browser-warning': '1' }
    })
    if (!res.ok) throw new Error()
    emit('saved', url)
  } catch {
    errorMsg.value = '❌ เชื่อมต่อไม่ได้ — ตรวจสอบว่า Colab กำลังรันอยู่'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.overlay {
  position: fixed; inset: 0;
  background: rgba(26,74,58,0.5);
  backdrop-filter: blur(6px);
  display: flex; align-items: center; justify-content: center;
  z-index: 200;
}

.modal {
  background: #fff;
  border-radius: 22px;
  padding: 32px 28px;
  width: min(440px, 92vw);
  box-shadow: 0 24px 64px rgba(26,74,58,0.22);
  display: flex; flex-direction: column; gap: 12px;
}

.modal-icon { font-size: 32px; text-align: center; }

h2 {
  font-family: 'Noto Serif Thai', serif;
  font-size: 20px;
  color: var(--green-deep);
  text-align: center;
}

p {
  font-size: 13.5px;
  color: var(--ink-soft);
  line-height: 1.65;
  text-align: center;
}

code {
  background: var(--green-pale);
  padding: 1px 6px;
  border-radius: 4px;
  font-size: 12px;
  color: var(--green-deep);
}

input {
  border: 1.5px solid var(--border);
  border-radius: 12px;
  padding: 11px 14px;
  font-family: 'Sarabun', sans-serif;
  font-size: 14px;
  color: var(--ink);
  background: var(--green-faint);
  outline: none;
  transition: border-color .2s;
}
input:focus   { border-color: var(--green-mid); background: #fff; }
input.error   { border-color: #ef4444; }

.error-msg { color: #dc2626; font-size: 12px; text-align: center; }

.btn-save {
  padding: 13px;
  background: var(--green-deep);
  color: #fff;
  border: none;
  border-radius: 12px;
  font-family: 'Sarabun', sans-serif;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: background .2s;
  display: flex; align-items: center; justify-content: center; gap: 8px;
  margin-top: 4px;
}
.btn-save:hover:not(:disabled) { background: var(--green-mid); }
.btn-save:disabled { opacity: .6; cursor: not-allowed; }

.spinner {
  width: 16px; height: 16px;
  border: 2px solid rgba(255,255,255,.4);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin .7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* transition */
.modal-enter-active, .modal-leave-active { transition: opacity .2s, transform .2s; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(.96); }
</style>
