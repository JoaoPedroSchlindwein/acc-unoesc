<template>
  <div class="stack-viz">
    <div class="controls">
      <input
        v-model.number="newValue"
        type="number"
        placeholder="Valor"
        @keyup.enter="push"
        class="form-input"
      />
      <button @click="push" class="btn btn-secondary">push()</button>
      <button @click="pop" class="btn btn-error" :disabled="stack.length === 0">
        pop()
      </button>
      <button @click="clear" class="btn btn-ghost">clear</button>
    </div>

    <div class="info">
      <span class="info-item">size(): {{ stack.length }}</span>
      <span class="info-item">empty(): {{ stack.length === 0 ? 'true' : 'false' }}</span>
      <span v-if="stack.length > 0" class="info-item">top(): {{ stack[stack.length - 1].val }}</span>
    </div>

    <div class="visualization">
      <div class="stack-container">
        <div class="stack-label">TOP (topo) ⬆</div>
        <transition-group name="stack" class="stack-items">
          <div
            v-for="(value, index) in [...stack].reverse()"
            :key="value.id"
            class="stack-cell"
            :class="{ top: index === 0 }"
          >
            {{ value.val }}
          </div>
        </transition-group>
        <div class="stack-base">BASE</div>
      </div>

      <div class="lifo-explanation">
        <strong>LIFO:</strong> Last In, First Out
        <br />
        O último elemento inserido é o primeiro a sair.
        <br />
        <br />
        <strong>Aplicações:</strong>
        <ul>
          <li>Inversão de sequências</li>
          <li>Validação de parênteses</li>
          <li>Pilha de chamadas de funções</li>
        </ul>
      </div>
    </div>

    <div class="code-display">
      <pre><code>{{ currentCode }}</code></pre>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const stack = ref([
  { id: 1, val: 10 },
  { id: 2, val: 20 },
  { id: 3, val: 30 }
])
const newValue = ref(40)
const currentCode = ref('// Stack LIFO inicializada com [10, 20, 30]')
let nextId = 4

const push = () => {
  if (newValue.value !== null && newValue.value !== '') {
    stack.value.push({ id: nextId++, val: newValue.value })
    currentCode.value = `s.push(${newValue.value}); // O(1)\n// Inserido no TOPO\n// size: ${stack.value.length}`
    newValue.value = newValue.value + 10
  }
}

const pop = () => {
  if (stack.value.length > 0) {
    const removed = stack.value.pop()
    currentCode.value = `s.pop(); // Removeu ${removed.val} do TOPO\n// size: ${stack.value.length}`
  }
}

const clear = () => {
  stack.value = []
  currentCode.value = '// Stack esvaziada'
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
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: var(--spacing-xl);
  align-items: start;
}

.stack-container {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-sm);
}

.stack-label {
  text-align: center;
  font-weight: var(--font-weight-semibold);
  color: var(--color-error-600);
  font-size: var(--text-sm);
  padding: var(--spacing-sm);
  background: var(--color-error-50);
  border-radius: var(--radius-md);
}

.stack-items {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
  min-height: 200px;
  padding: var(--spacing-md);
  background: var(--color-background-alt);
  border-radius: var(--radius-md);
}

.stack-cell {
  width: 100%;
  height: 60px;
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

.stack-cell.top {
  background: var(--color-error-500);
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.stack-base {
  text-align: center;
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-secondary);
  font-size: var(--text-sm);
  padding: var(--spacing-sm);
  background: var(--color-neutral-200);
  border-radius: var(--radius-md);
}

.stack-enter-active,
.stack-leave-active {
  transition: all 0.4s ease;
}

.stack-enter-from,
.stack-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

.lifo-explanation {
  padding: var(--spacing-lg);
  background: var(--color-secondary-50);
  border-left: 4px solid var(--color-secondary-500);
  border-radius: var(--radius-sm);
  line-height: 1.6;
}

.lifo-explanation ul {
  margin-top: var(--spacing-sm);
  margin-left: var(--spacing-lg);
}

.lifo-explanation li {
  margin: var(--spacing-xs) 0;
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

@media (max-width: 768px) {
  .visualization {
    grid-template-columns: 1fr;
  }
}
</style>