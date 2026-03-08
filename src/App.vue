<template>
  <div class="app">

    <!-- Header -->
    <header>
      <div class="header-icon">🩺</div>
      <div class="header-text">
        <h1>ผู้ช่วยแพทย์ AI</h1>
        <p>Nutella001 · Llama 3.2 3B</p>
      </div>
      <div class="header-actions">
        <div class="status-pill" :class="{ online: isOnline, offline: !isOnline }">
          <span class="dot"></span>
          {{ isOnline ? 'พร้อมใช้งาน' : 'ไม่ได้เชื่อมต่อ' }}
        </div>
        <button class="icon-btn" title="ตั้งค่า API" @click="showModal = true">⚙️</button>
      </div>
    </header>

    <!-- Disclaimer -->
    <div class="disclaimer">
      ⚠️ ข้อมูลนี้ใช้เพื่อประกอบการตัดสินใจเบื้องต้นเท่านั้น ไม่สามารถแทนที่การวินิจฉัยจากแพทย์ได้
    </div>

    <!-- Messages -->
    <div class="messages" ref="messagesEl">

      <!-- Empty state -->
      <Transition name="fade">
        <div v-if="messages.length === 0" class="empty-state">
          <div class="empty-icon">🏥</div>
          <h2>สวัสดีครับ มีอาการอะไรไหม?</h2>
          <p>ถามเกี่ยวกับอาการ ยา หรือสุขภาพของคุณได้เลย<br/>ผมจะให้คำแนะนำเบื้องต้น</p>
          <div class="chips">
            <button
              v-for="s in suggestions" :key="s"
              class="chip"
              @click="sendSuggestion(s)"
            >{{ s }}</button>
          </div>
        </div>
      </Transition>

      <!-- Chat bubbles -->
      <ChatBubble v-for="(m, i) in messages" :key="i" :msg="m" />

      <!-- Typing -->
      <TypingIndicator v-if="loading" />
    </div>

    <!-- Input bar -->
    <div class="input-bar">
      <textarea
        ref="inputEl"
        v-model="inputText"
        placeholder="พิมอาการหรือคำถามที่นี่..."
        rows="1"
        @keydown.enter.exact.prevent="send"
        @input="autoResize"
      ></textarea>
      <button class="send-btn" :disabled="loading || !inputText.trim()" @click="send">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
          <line x1="22" y1="2" x2="11" y2="13"></line>
          <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
        </svg>
      </button>
    </div>

  </div>

  <!-- Modal -->
  <ApiModal
    :show="showModal"
    :initial-url="apiUrl"
    @saved="onApiSaved"
    @close="showModal = false"
  />
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'
import ChatBubble      from './components/ChatBubble.vue'
import TypingIndicator from './components/TypingIndicator.vue'
import ApiModal        from './components/ApiModal.vue'

// ── State ──
const apiUrl      = ref(localStorage.getItem('nutella_api_url') || '')
const isOnline    = ref(false)
const showModal   = ref(false)
const messages    = ref([])
const inputText   = ref('')
const loading     = ref(false)
const messagesEl  = ref(null)
const inputEl     = ref(null)

const suggestions = [
  'ปวดหัวบ่อยๆ ควรทำอย่างไร?',
  'มีไข้สูง 38 องศา ควรกินยาอะไร?',
  'เป็นหวัดมา 3 วัน ยังไม่หาย',
  'ปวดท้องหลังกินข้าว',
]

// ── Init ──
onMounted(() => {
  if (!apiUrl.value) {
    showModal.value = true
  } else {
    checkHealth()
    setInterval(checkHealth, 30000)
  }
})

// ── API ──
async function checkHealth() {
  try {
    const res = await fetch(`${apiUrl.value}/health`, {
      headers: { 'ngrok-skip-browser-warning': '1' }
    })
    isOnline.value = res.ok
  } catch {
    isOnline.value = false
  }
}

function onApiSaved(url) {
  apiUrl.value = url
  localStorage.setItem('nutella_api_url', url)
  showModal.value = false
  isOnline.value = true
  setInterval(checkHealth, 30000)
}

// ── Chat ──
function now() {
  return new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' })
}

function sendSuggestion(text) {
  inputText.value = text
  send()
}

async function send() {
  const text = inputText.value.trim()
  if (!text || loading.value) return
  if (!apiUrl.value) { showModal.value = true; return }

  messages.value.push({ role: 'user', content: text, time: now() })
  inputText.value = ''
  await nextTick()
  resetResize()
  scrollBottom()

  loading.value = true

  // Build history (exclude latest user msg)
  const history = messages.value.slice(0, -1).map(m => ({
    role: m.role === 'ai' ? 'assistant' : 'user',
    content: m.content,
  }))

  try {
    const res = await fetch(`${apiUrl.value}/chat`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'ngrok-skip-browser-warning': '1',
      },
      body: JSON.stringify({ message: text, history }),
    })

    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const data = await res.json()
    messages.value.push({ role: 'ai', content: data.response || '(ไม่มีคำตอบ)', time: now() })
    isOnline.value = true

  } catch (err) {
    messages.value.push({
      role: 'ai',
      content: `⚠️ เกิดข้อผิดพลาด: ${err.message}\nตรวจสอบว่า Colab กำลังรันอยู่`,
      time: now(),
    })
    isOnline.value = false
  }

  loading.value = false
  await nextTick()
  scrollBottom()
}

// ── Helpers ──
function scrollBottom() {
  if (messagesEl.value)
    messagesEl.value.scrollTop = messagesEl.value.scrollHeight
}

function autoResize(e) {
  const el = e.target
  el.style.height = 'auto'
  el.style.height = Math.min(el.scrollHeight, 120) + 'px'
}

function resetResize() {
  if (inputEl.value) inputEl.value.style.height = 'auto'
}
</script>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  max-width: 780px;
  margin: 0 auto;
  background: #fff;
  box-shadow: 0 0 60px rgba(26,74,58,.1);
}

/* ── Header ── */
header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px 22px;
  background: var(--green-deep);
  color: #fff;
  position: relative;
  overflow: hidden;
}
header::before {
  content: '';
  position: absolute;
  right: -40px; top: -40px;
  width: 180px; height: 180px;
  border-radius: 50%;
  background: rgba(255,255,255,.04);
  pointer-events: none;
}

.header-icon {
  width: 44px; height: 44px;
  background: var(--green-light);
  border-radius: 12px;
  display: flex; align-items: center; justify-content: center;
  font-size: 22px;
  flex-shrink: 0;
  box-shadow: 0 2px 8px rgba(0,0,0,.2);
}

.header-text h1 {
  font-family: 'Noto Serif Thai', serif;
  font-size: 17px;
  font-weight: 700;
}
.header-text p { font-size: 12px; opacity: .6; margin-top: 2px; }

.header-actions {
  margin-left: auto;
  display: flex;
  align-items: center;
  gap: 10px;
}

.status-pill {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  background: rgba(255,255,255,.12);
  padding: 4px 10px;
  border-radius: 999px;
}
.status-pill .dot {
  width: 7px; height: 7px;
  border-radius: 50%;
}
.status-pill.online  .dot { background: #6ee7b7; animation: pulse 2s infinite; }
.status-pill.offline .dot { background: #f87171; }

@keyframes pulse {
  0%,100% { opacity:1; }
  50%     { opacity:.35; }
}

.icon-btn {
  background: rgba(255,255,255,.12);
  border: none;
  border-radius: 8px;
  width: 34px; height: 34px;
  font-size: 16px;
  cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  transition: background .2s;
}
.icon-btn:hover { background: rgba(255,255,255,.22); }

/* ── Disclaimer ── */
.disclaimer {
  background: #fffbeb;
  border-bottom: 1px solid #fde68a;
  padding: 8px 20px;
  font-size: 12px;
  color: #92400e;
}

/* ── Messages ── */
.messages {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  scroll-behavior: smooth;
}

/* ── Empty state ── */
.empty-state {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  gap: 10px;
  color: var(--ink-soft);
  padding: 40px 20px;
}
.empty-icon { font-size: 52px; margin-bottom: 6px; }
.empty-state h2 {
  font-family: 'Noto Serif Thai', serif;
  font-size: 19px;
  color: var(--green-deep);
}
.empty-state p { font-size: 14px; line-height: 1.65; }

.chips {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  margin-top: 16px;
}
.chip {
  padding: 8px 15px;
  background: var(--green-pale);
  border: 1px solid var(--border);
  border-radius: 999px;
  font-size: 13px;
  color: var(--green-deep);
  font-family: 'Sarabun', sans-serif;
  cursor: pointer;
  transition: background .2s, transform .1s;
}
.chip:hover { background: var(--border); transform: translateY(-1px); }

/* ── Input bar ── */
.input-bar {
  padding: 12px 16px;
  background: #fff;
  border-top: 1px solid var(--border);
  display: flex;
  gap: 10px;
  align-items: flex-end;
}

textarea {
  flex: 1;
  resize: none;
  border: 1.5px solid var(--border);
  border-radius: 12px;
  padding: 11px 14px;
  font-family: 'Sarabun', sans-serif;
  font-size: 14.5px;
  color: var(--ink);
  background: var(--green-faint);
  outline: none;
  max-height: 120px;
  min-height: 44px;
  line-height: 1.5;
  transition: border-color .2s, background .2s;
}
textarea:focus     { border-color: var(--green-mid); background: #fff; }
textarea::placeholder { color: #9ab5ab; }

.send-btn {
  width: 44px; height: 44px;
  border-radius: 12px;
  border: none;
  background: var(--green-deep);
  color: #fff;
  cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
  transition: background .2s, transform .1s;
}
.send-btn svg { width: 18px; height: 18px; }
.send-btn:hover:not(:disabled) { background: var(--green-mid); transform: scale(1.06); }
.send-btn:disabled { background: var(--border); cursor: not-allowed; }

/* transition */
.fade-enter-active, .fade-leave-active { transition: opacity .3s; }
.fade-enter-from, .fade-leave-to       { opacity: 0; }
</style>