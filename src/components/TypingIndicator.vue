<template>
  <div class="bubble-row">
    <div class="avatar ai">🩺</div>
    <div class="bubble ai">
      <span class="dot"></span>
      <span class="dot"></span>
      <span class="dot"></span>
    </div>
  </div>
</template>

<style scoped>
.bubble-row {
  display: flex;
  gap: 10px;
  align-items: flex-end;
  animation: fadeUp .2s ease both;
}
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.avatar {
  width: 34px; height: 34px;
  border-radius: 50%;
  background: var(--green-pale);
  border: 1px solid var(--border);
  display: flex; align-items: center; justify-content: center;
  font-size: 16px; flex-shrink: 0;
}

.bubble {
  display: flex;
  gap: 5px;
  align-items: center;
  padding: 14px 18px;
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 16px;
  border-bottom-left-radius: 4px;
  box-shadow: 0 2px 12px rgba(26,74,58,.07);
}

.dot {
  width: 7px; height: 7px;
  border-radius: 50%;
  background: var(--green-light);
  animation: bounce 1.2s infinite;
}
.dot:nth-child(2) { animation-delay: .2s; }
.dot:nth-child(3) { animation-delay: .4s; }

@keyframes bounce {
  0%, 80%, 100% { transform: translateY(0); opacity: .5; }
  40%            { transform: translateY(-6px); opacity: 1; }
}
</style>