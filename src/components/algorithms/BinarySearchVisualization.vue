<template>
  <div class="binary-search-viz">
    <div class="controls">
      <label>
        Buscar valor:
        <input v-model.number="target" type="number" class="form-input" />
      </label>
      <button @click="startSearch" :disabled="isSearching" class="btn btn-primary">
        Buscar
      </button>
      <button @click="reset" class="btn btn-ghost">Reset</button>
    </div>

    <div class="info">
      <div v-if="!isSearching && found !== null">
        <span v-if="found" class="info-item found-msg">
          ✓ Elemento {{ target }} encontrado no índice {{ foundIndex }}
        </span>
        <span v-else class="info-item not-found">✗ Elemento {{ target }} não encontrado</span>
      </div>
      <div v-if="isSearching" class="info-searching">
        <span class="info-item">Iteração: {{ iteration }}</span>
        <span class="info-item">Espaço de busca: {{ searchSpace }} elementos</span>
      </div>
    </div>

    <div class="visualization">
      <div class="array-container">
        <div
          v-for="(value, index) in array"
          :key="index"
          class="array-cell"
          :class="{
            left: index === left && isSearching,
            right: index === right && isSearching,
            mid: index === mid && isSearching,
            found: index === foundIndex && !isSearching,
            eliminated: (index < left || index > right) && isSearching
          }"
        >
          <div class="index">{{ index }}</div>
          <div class="value">{{ value }}</div>
          <div v-if="index === left && isSearching" class="pointer left-pointer">L</div>
          <div v-if="index === right && isSearching" class="pointer right-pointer">R</div>
          <div v-if="index === mid && isSearching" class="pointer mid-pointer">M</div>
        </div>
      </div>

      <div v-if="isSearching" class="step-explanation">
        <strong>Passo {{ iteration }}:</strong>
        <div>left = {{ left }}, right = {{ right }}, mid = {{ mid }}</div>
        <div>array[{{ mid }}] = {{ array[mid] }}</div>
        <div>
          {{ array[mid] === target ? '✓ Encontrado!' :
             array[mid] < target ? `${array[mid]} < ${target} → buscar na direita` :
             `${array[mid]} > ${target} → buscar na esquerda` }}
        </div>
      </div>
    </div>

    <div class="complexity-info">
      <h4>Complexidade</h4>
      <p><strong>Tempo:</strong> O(log n) - Cada iteração reduz o espaço pela metade</p>
      <p><strong>Espaço:</strong> O(1) - Apenas variáveis para índices</p>
      <p>
        Para n = {{ array.length }}, são necessárias no máximo
        {{ Math.ceil(Math.log2(array.length)) }} comparações
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const array = ref([2, 5, 8, 12, 16, 23, 38, 45, 56, 67, 78, 89, 95])
const target = ref(23)
const left = ref(0)
const right = ref(array.value.length - 1)
const mid = ref(0)
const isSearching = ref(false)
const found = ref(null)
const foundIndex = ref(-1)
const iteration = ref(0)
const searchSpace = ref(array.value.length)

const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const startSearch = async () => {
  reset()
  isSearching.value = true
  found.value = null
  foundIndex.value = -1

  left.value = 0
  right.value = array.value.length - 1

  while (left.value <= right.value) {
    iteration.value++
    mid.value = Math.floor((left.value + right.value) / 2)
    searchSpace.value = right.value - left.value + 1

    await delay(1500)

    if (array.value[mid.value] === target.value) {
      found.value = true
      foundIndex.value = mid.value
      isSearching.value = false
      return
    }

    if (array.value[mid.value] < target.value) {
      left.value = mid.value + 1
    } else {
      right.value = mid.value - 1
    }

    await delay(500)
  }

  found.value = false
  isSearching.value = false
}

const reset = () => {
  isSearching.value = false
  found.value = null
  foundIndex.value = -1
  iteration.value = 0
  left.value = 0
  right.value = array.value.length - 1
  mid.value = 0
  searchSpace.value = array.value.length
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

.controls label {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  font-weight: var(--font-weight-medium);
}

.form-input {
  padding: var(--spacing-sm) var(--spacing-md);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: var(--text-base);
  width: 100px;
}

.info {
  margin-bottom: var(--spacing-lg);
  min-height: 30px;
  font-family: var(--font-mono);
}

.info-searching {
  display: flex;
  gap: var(--spacing-lg);
}

.info-item {
  background: var(--color-neutral-100);
  color: var(--color-text-secondary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-md);
  display: inline-block;
}

.found-msg {
  background-color: var(--color-secondary-100);
  color: var(--color-secondary-800);
}

.not-found {
  background-color: var(--color-error-100);
  color: var(--color-error-700);
}

.visualization {
  background: var(--color-surface);
  padding: var(--spacing-xl);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  margin-bottom: var(--spacing-lg);
}

.array-container {
  display: flex;
  gap: var(--spacing-xs);
  margin-bottom: var(--spacing-xl);
  overflow-x: auto;
  padding: calc(var(--spacing-xl) + var(--spacing-sm)) var(--spacing-sm);
}

.array-cell {
  min-width: 60px;
  height: 80px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  background: var(--color-surface);
  transition: all 0.4s ease;
}

.array-cell.eliminated {
  opacity: 0.3;
  background: var(--color-neutral-100);
}

.array-cell.left {
  border-color: var(--color-primary-500);
  background: var(--color-primary-50);
}

.array-cell.right {
  border-color: var(--color-error-500);
  background: var(--color-error-50);
}

.array-cell.mid {
  border-color: var(--color-accent-500);
  background: var(--color-accent-50);
  transform: scale(1.1);
}

.array-cell.found {
  border-color: var(--color-secondary-500);
  background: var(--color-secondary-100);
  transform: scale(1.15);
  animation: found 0.6s ease;
}

@keyframes found {
  0%, 100% { transform: scale(1.15); }
  50% { transform: scale(1.25); }
}

.index {
  position: absolute;
  top: calc(-1 * var(--spacing-lg));
  font-size: var(--text-xs);
  color: var(--color-text-tertiary);
}

.value {
  font-size: var(--text-lg);
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-primary);
  font-family: var(--font-mono);
}

.pointer {
  position: absolute;
  bottom: calc(-1 * var(--spacing-xl));
  font-size: var(--text-sm);
  font-weight: var(--font-weight-bold);
  padding: var(--spacing-xs) var(--spacing-sm);
  border-radius: var(--radius-sm);
}

.left-pointer {
  color: var(--color-primary-700);
  background: var(--color-primary-100);
}

.right-pointer {
  color: var(--color-error-700);
  background: var(--color-error-100);
}

.mid-pointer {
  color: var(--color-accent-700);
  background: var(--color-accent-100);
}

.step-explanation {
  background: var(--color-background-alt);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  border-left: 4px solid var(--color-secondary-500);
  line-height: 1.6;
}

.step-explanation div {
  font-family: var(--font-mono);
  margin: var(--spacing-sm) 0;
}

.complexity-info {
  background: var(--color-primary-50);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  border-left: 4px solid var(--color-primary-500);
}

.complexity-info h4 {
  color: var(--color-text-primary);
  margin-bottom: var(--spacing-sm);
}

.complexity-info p {
  margin: 0.5rem 0;
  line-height: 1.6;
  color: var(--color-text-secondary);
}
</style>