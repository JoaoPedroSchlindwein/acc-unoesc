<template>
  <div class="two-pointers-viz">
    <div class="controls">
      <label>
        Soma alvo:
        <input v-model.number="targetSum" type="number" class="form-input" />
      </label>
      <button @click="findPair" :disabled="isSearching" class="btn btn-primary">
        Buscar Par
      </button>
      <button @click="reset" class="btn btn-ghost">Reset</button>
    </div>

    <div class="info">
      <div v-if="!isSearching && found !== null">
        <span v-if="found" class="info-item found-msg">
          ✓ Par encontrado: {{ array[foundLeft] }} + {{ array[foundRight] }} = {{ targetSum }}
        </span>
        <span v-else class="info-item not-found">
          ✗ Nenhum par com soma {{ targetSum }}
        </span>
      </div>
      <div v-if="isSearching" class="info-item">
        <span>Soma atual: {{ array[left] }} + {{ array[right] }} = {{ array[left] + array[right] }}</span>
      </div>
    </div>

    <div class="visualization">
      <div class="pattern-selector">
        <h4>Padrão:</h4>
        <button
          @click="pattern = 'convergence'"
          :class="{ active: pattern === 'convergence' }"
          class="btn btn-sm btn-outline"
        >
          Convergência
        </button>
        <button
          @click="pattern = 'chase'"
          :class="{ active: pattern === 'chase' }"
          class="btn btn-sm btn-outline"
        >
          Perseguição
        </button>
      </div>

      <div class="array-container">
        <div
          v-for="(value, index) in array"
          :key="index"
          class="array-cell"
          :class="{
            left: index === left && isSearching,
            right: index === right && isSearching,
            found: (index === foundLeft || index === foundRight) && !isSearching,
            eliminated: (index < left || index > right) && isSearching && pattern === 'convergence'
          }"
        >
          <div class="index">{{ index }}</div>
          <div class="value">{{ value }}</div>
          <div v-if="index === left && isSearching" class="pointer left-pointer">
            {{ pattern === 'convergence' ? 'LEFT' : 'i' }}
          </div>
          <div v-if="index === right && isSearching" class="pointer right-pointer">
            {{ pattern === 'convergence' ? 'RIGHT' : 'j' }}
          </div>
        </div>
      </div>

      <div v-if="isSearching" class="step-explanation">
        <strong>Estado atual:</strong>
        <div>left = {{ left }}, right = {{ right }}</div>
        <div>array[{{ left }}] + array[{{ right }}] = {{ array[left] }} + {{ array[right] }} = {{ array[left] + array[right] }}</div>
        <div>
          {{ array[left] + array[right] === targetSum ? '✓ Soma igual ao alvo!' :
             array[left] + array[right] < targetSum ? `Soma muito pequena → mover LEFT para a direita` :
             `Soma muito grande → mover RIGHT para a esquerda` }}
        </div>
      </div>
    </div>

    <div class="explanation-section">
      <div class="explanation">
        <h4>Padrão de Convergência</h4>
        <p>
          Ponteiros começam nas extremidades e se movem em direção ao centro.
          Ideal para problemas em arrays ordenados.
        </p>
        <p><strong>Exemplo:</strong> Encontrar par com soma específica</p>
      </div>

      <div class="explanation">
        <h4>Padrão de Perseguição</h4>
        <p>
          Ambos os ponteiros se movem na mesma direção, mantendo uma janela válida.
        </p>
        <p><strong>Exemplo:</strong> Remover duplicatas, subarrays com soma específica</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const array = ref([2, 5, 8, 12, 16, 23, 38, 45, 56])
const targetSum = ref(50)
const left = ref(0)
const right = ref(array.value.length - 1)
const isSearching = ref(false)
const found = ref(null)
const foundLeft = ref(-1)
const foundRight = ref(-1)
const pattern = ref('convergence')

const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const findPair = async () => {
  reset()
  isSearching.value = true
  found.value = null

  left.value = 0
  right.value = array.value.length - 1

  while (left.value < right.value) {
    await delay(1200)

    const sum = array.value[left.value] + array.value[right.value]

    if (sum === targetSum.value) {
      found.value = true
      foundLeft.value = left.value
      foundRight.value = right.value
      isSearching.value = false
      return
    }

    if (sum < targetSum.value) {
      left.value++
    } else {
      right.value--
    }

    await delay(400)
  }

  found.value = false
  isSearching.value = false
}

const reset = () => {
  isSearching.value = false
  found.value = null
  foundLeft.value = -1
  foundRight.value = -1
  left.value = 0
  right.value = array.value.length - 1
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
}

.info-item {
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-md);
  display: inline-block;
  margin-bottom: var(--spacing-sm);
  font-family: var(--font-mono);
}

.found-msg {
  background: var(--color-secondary-100);
  color: var(--color-secondary-800);
}

.not-found {
  background: var(--color-error-100);
  color: var(--color-error-700);
}

.visualization {
  background: var(--color-surface);
  padding: var(--spacing-xl);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  margin-bottom: var(--spacing-lg);
}

.pattern-selector {
  margin-bottom: var(--spacing-xl);
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
}

.pattern-selector h4 {
  margin: 0;
  color: var(--color-text-secondary);
}

.pattern-selector .btn-outline.active {
  background-color: var(--color-primary-600);
  border-color: var(--color-primary-600);
  color: var(--color-text-inverse);
}

.array-container {
  display: flex;
  gap: var(--spacing-xs);
  margin-bottom: var(--spacing-xl);
  overflow-x: auto;
  padding: calc(var(--spacing-xl) + var(--spacing-sm)) var(--spacing-sm);
}

.array-cell {
  min-width: 70px;
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
  transform: scale(1.1);
}

.array-cell.right {
  border-color: var(--color-error-500);
  background: var(--color-error-50);
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
  font-size: var(--text-xs);
  font-weight: var(--font-weight-bold);
  padding: var(--spacing-xs) var(--spacing-sm);
  border-radius: var(--radius-sm);
  white-space: nowrap;
}

.left-pointer {
  color: var(--color-primary-700);
  background: var(--color-primary-100);
}

.right-pointer {
  color: var(--color-error-700);
  background: var(--color-error-100);
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

.explanation-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: var(--spacing-lg);
}

.explanation {
  background: var(--color-primary-50);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  border-left: 4px solid var(--color-primary-500);
}

.explanation h4 {
  color: var(--color-text-primary);
  margin-bottom: var(--spacing-sm);
}

.explanation p {
  margin: 0.5rem 0;
  line-height: 1.6;
  color: var(--color-text-secondary);
}
</style>