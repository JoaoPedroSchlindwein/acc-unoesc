<template>
  <div class="queue-viz">
    <div class="controls">
      <input
        v-model.number="newValue"
        type="number"
        placeholder="Valor"
        @keyup.enter="push"
        class="form-input"
      />
      <button @click="push" class="btn btn-secondary">push()</button>
      <button @click="pop" class="btn btn-error" :disabled="queue.length === 0">
        pop()
      </button>
      <button @click="clear" class="btn btn-ghost">clear</button>
    </div>

    <div class="info">
      <span class="info-item">size(): {{ queue.length }}</span>
      <span class="info-item">empty(): {{ queue.length === 0 ? 'true' : 'false' }}</span>
      <span v-if="queue.length > 0" class="info-item">front(): {{ queue[0].val }}</span>
      <span v-if="queue.length > 0" class="info-item">back(): {{ queue[queue.length - 1].val }}</span>
    </div>

    <div class="visualization">
      <div class="queue-label">FRONT (saída)</div>
      <div class="queue-container">
        <transition-group name="queue">
          <div
            v-for="(value, index) in queue"
            :key="value.id"
            class="queue-cell"
            :class="{ first: index === 0, last: index === queue.length - 1 }"
          >
            {{ value.val }}
          </div>
        </transition-group>
      </div>
      <div class="queue-label">BACK (entrada)</div>

      <div class="fifo-explanation">
        <strong>FIFO:</strong> First In, First Out
        <br />
        O primeiro elemento inserido é o primeiro a sair.
      </div>
    </div>

    <div class="code-display">
      <pre><code>{{ currentCode }}</code></pre>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const queue = ref([
  { id: 1, val: 10 },
  { id: 2, val: 20 },
  { id: 3, val: 30 }
])
const newValue = ref(40)
const currentCode = ref('// Queue FIFO inicializada com [10, 20, 30]')
let nextId = 4

const push = () => {
  if (newValue.value !== null && newValue.value !== '') {
    queue.value.push({ id: nextId++, val: newValue.value })
    currentCode.value = `q.push(${newValue.value}); // O(1)\n// Inserido no BACK\n// size: ${queue.value.length}`
    newValue.value = newValue.value + 10
  }
}

const pop = () => {
  if (queue.value.length > 0) {
    const removed = queue.value.shift()
    currentCode.value = `q.pop(); // Removeu ${removed.val} do FRONT\n// size: ${queue.value.length}`
  }
}

const clear = () => {
  queue.value = []
  currentCode.value = '// Queue esvaziada'
}
</script>

<style scoped>
.controls {
  display: flex;
  gap: var(--spacing-md);
  margin-bottom: var(--spacing-lg);
  align-items: center;
  flex-wrap: wrap;
}

.form-input {
  padding: var(--spacing-sm) var(--spacing-md);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: var(--text-base);
  width: 120px;
  transition: border-color var(--transition-base);
}

.form-input:focus {
  border-color: var(--color-primary-500);
  outline: none;
}

.info {
  display: flex;
  gap: var(--spacing-md);
  margin-bottom: var(--spacing-lg);
  font-family: var(--font-mono);
  font-size: var(--text-sm);
  flex-wrap: wrap;
}

.info-item {
  background: var(--color-neutral-100);
  color: var(--color-text-secondary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-md);
}

.visualization {
  background: var(--color-surface);
  padding: var(--spacing-xl);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  margin-bottom: var(--spacing-lg);
}

.queue-label {
  text-align: center;
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-secondary);
  margin: var(--spacing-sm) 0;
  font-size: var(--text-sm);
}

.queue-container {
  display: flex;
  gap: var(--spacing-sm);
  padding: var(--spacing-md);
  background: var(--color-background-alt);
  border-radius: var(--radius-md);
  min-height: 100px;
  align-items: center;
  overflow-x: auto;
}

.queue-cell {
  min-width: 80px;
  height: 80px;
  background: var(--color-secondary-500);
  color: var(--color-text-inverse);
  border-radius: var(--radius-md);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: var(--text-xl);
  font-weight: var(--font-weight-bold);
  font-family: var(--font-mono);
  box-shadow: var(--shadow-base);
}

.queue-cell.first {
  background: var(--color-error-500);
  animation: pulse 1.5s infinite;
}

.queue-cell.last {
  background: var(--color-primary-500);
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.queue-enter-active,
.queue-leave-active {
  transition: all 0.5s ease;
}

.queue-enter-from {
  opacity: 0;
  transform: translateX(30px);
}

.queue-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

.fifo-explanation {
  margin-top: var(--spacing-lg);
  padding: var(--spacing-lg);
  background: var(--color-secondary-50);
  border-left: 4px solid var(--color-secondary-500);
  border-radius: var(--radius-sm);
  line-height: 1.6;
}

.code-display {
  background: var(--color-neutral-900);
  color: var(--color-neutral-100);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  font-family: var(--font-mono);
  font-size: var(--text-sm);
}

.code-display pre {
  margin: 0;
}
</style>