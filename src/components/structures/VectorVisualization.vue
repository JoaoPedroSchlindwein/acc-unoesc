<template>
  <div class="vector-viz">
    <div class="controls">
      <input
        v-model.number="newValue"
        type="number"
        placeholder="Valor"
        @keyup.enter="pushBack"
        class="form-input"
      />
      <button @click="pushBack" class="btn btn-secondary">
        push_back()
      </button>
      <button @click="popBack" class="btn btn-error" :disabled="vector.length === 0">
        pop_back()
      </button>
      <button @click="clear" class="btn btn-ghost">clear()</button>
    </div>

    <div class="info">
      <span class="info-item">size(): {{ vector.length }}</span>
      <span class="info-item">capacity(): {{ capacity }}</span>
      <span v-if="vector.length > 0" class="info-item">front(): {{ vector[0] }}</span>
      <span v-if="vector.length > 0" class="info-item">back(): {{ vector[vector.length - 1] }}</span>
    </div>

    <div class="visualization">
      <div class="vector-container">
        <div
          v-for="(value, index) in capacity"
          :key="index"
          class="vector-cell"
          :class="{ filled: index < vector.length, highlighted: index === highlightIndex }"
        >
          <div class="index-label">{{ index }}</div>
          <div class="value">{{ index < vector.length ? vector[index] : '' }}</div>
        </div>
      </div>
    </div>

    <div class="code-display">
      <pre><code>{{ currentCode }}</code></pre>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const vector = ref([10, 20, 30])
const newValue = ref(40)
const highlightIndex = ref(-1)
const currentCode = ref('// Vector inicializado com [10, 20, 30]')

const capacity = computed(() => {
  if (vector.value.length === 0) return 0
  let cap = 1
  while (cap < vector.value.length) cap *= 2
  return Math.max(cap, 8)
})

const pushBack = () => {
  if (newValue.value !== null && newValue.value !== '') {
    vector.value.push(newValue.value)
    highlightIndex.value = vector.value.length - 1
    currentCode.value = `v.push_back(${newValue.value}); // O(1) amortizado\n// size: ${vector.value.length}`

    setTimeout(() => {
      highlightIndex.value = -1
    }, 1000)

    newValue.value = newValue.value + 10
  }
}

const popBack = () => {
  if (vector.value.length > 0) {
    const removed = vector.value.pop()
    currentCode.value = `v.pop_back(); // Removeu ${removed}\n// size: ${vector.value.length}`
  }
}

const clear = () => {
  vector.value = []
  currentCode.value = 'v.clear(); // size: 0'
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

.vector-container {
  display: flex;
  gap: var(--spacing-xs);
  overflow-x: auto;
  padding: calc(var(--spacing-lg) + var(--spacing-xs)) var(--spacing-xs);
}

.vector-cell {
  min-width: 80px;
  height: 80px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  background: var(--color-background-alt);
  transition: all var(--transition-slow);
}

.vector-cell.filled {
  background: var(--color-surface);
  border-color: var(--color-secondary-500);
}

.vector-cell.highlighted {
  background: var(--color-accent-100);
  border-color: var(--color-accent-500);
  transform: scale(1.05);
}

.index-label {
  position: absolute;
  top: calc(-1 * var(--spacing-lg));
  font-size: var(--text-xs);
  color: var(--color-text-tertiary);
}

.value {
  font-size: var(--text-xl);
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-primary);
  font-family: var(--font-mono);
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