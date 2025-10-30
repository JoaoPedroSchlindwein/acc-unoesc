<template>
  <div class="binary-search-enhanced">
    <!-- Controles -->
    <div class="controls-grid">
      <div class="control-section quick-tests">
        <h4>✨ Casos de Teste</h4>
        <div class="button-group">
          <button @click="loadTest('simple')" class="btn btn-sm btn-outline">Simples</button>
          <button @click="loadTest('large')" class="btn btn-sm btn-outline">Grande</button>
          <button @click="loadTest('notfound')" class="btn btn-sm btn-outline">Não Encontrado</button>
          <button @click="loadTest('single')" class="btn btn-sm btn-outline">1 Elemento</button>
          <button @click="loadTest('empty')" class="btn btn-sm btn-outline">Vazio</button>
        </div>
      </div>

      <div class="control-section main-controls">
        <h4>
          
          Controles</h4>
        <div class="input-group">
          <label for="target-input">Buscar valor:</label>
          <input id="target-input" v-model.number="target" type="number" :disabled="isPlaying" class="form-input" />
          <button @click="initialize" :disabled="isPlaying || initialized" class="btn btn-primary">
            🚀 Iniciar
          </button>
        </div>
        <div v-if="initialized" class="player-controls">
          <button @click="previousStep" :disabled="currentStep === 0" class="btn btn-ghost">⏮</button>
          <button @click="togglePlay" class="btn btn-secondary btn-play">{{ isPlaying ? '⏸' : '▶' }}</button>
          <button @click="nextStep" :disabled="currentStep >= steps.length - 1" class="btn btn-ghost">
            ⨯
          </button>
          <button @click="reset" class="btn btn-error">
            ↩️
          </button>
        </div>
      </div>

      <div v-if="initialized" class="control-section speed-control">
        <h4>
          
          Velocidade</h4>
        <input
          v-model.number="speed"
          type="range"
          min="200"
          max="2000"
          step="200"
          class="speed-slider"
        />
        <span class="speed-label">{{ getSpeedLabel() }}</span>
      </div>
    </div>

    <!-- Timeline -->
    <div v-if="initialized && steps.length > 0" class="timeline-section">
      <div class="timeline-header">
        <span>Passo {{ currentStep + 1 }} de {{ steps.length }}</span>
        <span>{{ getStepDescription() }}</span>
      </div>
      <input
        v-model.number="currentStep"
        type="range"
        min="0"
        :max="steps.length - 1"
        class="timeline-slider"
        @input="onTimelineChange"
      />
    </div>

    <!-- Status & Visualization -->
    <div v-if="initialized" class="main-content-grid">
      <div class="status-info">
        <div class="stat-card">
          <div class="stat-label">Comparações</div>
          <div class="stat-value">{{ currentStepData ? currentStepData.iteration : 0 }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Espaço de Busca</div>
          <div class="stat-value">{{ getCurrentSearchSpace() }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Complexidade</div>
          <div class="stat-value">O(log n)</div>
        </div>
        <div v-if="currentStepData" class="stat-card" :class="getResultClass()">
          <div class="stat-label">Status</div>
          <div class="stat-value">{{ getStatus() }}</div>
        </div>
      </div>

      <div class="visualization">
        <div class="array-container">
          <div
            v-for="(value, index) in array"
            :key="index"
            class="array-cell"
            :class="getCellClass(index)"
          >
            <div class="index">{{ index }}</div>
            <div class="value">{{ value }}</div>
            <div v-if="showPointer(index, 'left')" class="pointer left-pointer">L</div>
            <div v-if="showPointer(index, 'right')" class="pointer right-pointer">R</div>
            <div v-if="showPointer(index, 'mid')" class="pointer mid-pointer">M</div>
          </div>
        </div>
      </div>
    </div>

    <!-- Explanation & Code -->
    <div v-if="initialized" class="details-grid">
      <div v-if="currentStepData" class="contextual-explanation">
        <div class="explanation-header">ℹ️ O que está acontecendo:</div>
        <div class="explanation-content" v-html="currentStepData.explanation"></div>
        <div v-if="currentStepData.tip" class="explanation-tip"><strong>Dica:</strong> {{ currentStepData.tip }}</div>
      </div>

      <div class="code-view">
        <h4>
          
          Código (linha atual destacada)</h4>
        <pre><code><span v-for="(line, idx) in codeLines" :key="idx" :class="{ highlighted: idx === currentCodeLine }">{{ line }}</span></code></pre>
      </div>
    </div>

  </div>
</template>

<script setup>
// ... (script content remains the same)
import { ref, computed, watch } from 'vue'

// State
const array = ref([2, 5, 8, 12, 16, 23, 38, 45, 56, 67, 78, 89, 95])
const target = ref(45)
const speed = ref(1000)
const initialized = ref(false)
const isPlaying = ref(false)
const currentStep = ref(0)
const steps = ref([])
const playInterval = ref(null)

// Code lines for highlighting
const codeLines = [
  'int buscaBinaria(vector<int>& v, int alvo) {',
  '    int esq = 0;',
  '    int dir = v.size() - 1;',
  '    ',
  '    while (esq <= dir) {',
  '        int meio = esq + (dir - esq) / 2;',
  '        ',
  '        if (v[meio] == alvo) {',
  '            return meio;  // Encontrado!',
  '        }',
  '        ',
  '        if (v[meio] < alvo) {',
  '            esq = meio + 1;  // Buscar à direita',
  '        } else {',
  '            dir = meio - 1;  // Buscar à esquerda',
  '        }',
  '    }',
  '    ',
  '    return -1;  // Não encontrado',
  '}'
]

const currentCodeLine = computed(() => {
  if (!currentStepData.value) return -1
  return currentStepData.value.codeLine
})

const currentStepData = computed(() => {
  return steps.value[currentStep.value]
})

// Quick test presets
const loadTest = (testType) => {
  reset()
  switch (testType) {
    case 'simple':
      array.value = [2, 5, 8, 12, 16, 23, 38, 45, 56, 67, 78, 89, 95]
      target.value = 45
      break
    case 'empty':
      array.value = []
      target.value = 5
      break
    case 'single':
      array.value = [42]
      target.value = 42
      break
    case 'large':
      array.value = Array.from({ length: 100 }, (_, i) => i * 2)
      target.value = 84
      break
    case 'notfound':
      array.value = [1, 3, 5, 7, 9, 11, 13, 15]
      target.value = 10
      break
  }
}

// Initialize and generate steps
const initialize = () => {
  if (array.value.length === 0) {
    alert('Array está vazio! Escolha um caso de teste.')
    return
  }

  steps.value = generateSteps()
  currentStep.value = 0
  initialized.value = true
}

const generateSteps = () => {
  const stepsArray = []
  let left = 0
  let right = array.value.length - 1
  let iteration = 0

  while (left <= right) {
    const mid = Math.floor((left + right) / 2)
    const searchSpace = right - left + 1

    stepsArray.push({
      left,
      right,
      mid,
      iteration: iteration + 1,
      searchSpace,
      found: array.value[mid] === target.value,
      comparison: array.value[mid] < target.value ? 'menor' : array.value[mid] > target.value ? 'maior' : 'igual',
      explanation: generateExplanation(left, right, mid, array.value[mid], target.value),
      tip: generateTip(searchSpace, iteration),
      codeLine: array.value[mid] === target.value ? 8 : (array.value[mid] < target.value ? 12 : 14)
    })

    if (array.value[mid] === target.value) {
      break
    }

    if (array.value[mid] < target.value) {
      left = mid + 1
    } else {
      right = mid - 1
    }

    iteration++
  }

  // Add final state if not found
  if (left > right && !stepsArray[stepsArray.length - 1]?.found) {
    stepsArray.push({
      left,
      right,
      mid: -1,
      iteration: iteration + 1,
      searchSpace: 0,
      found: false,
      comparison: 'notfound',
      explanation: `<p>O espaço de busca foi esgotado!</p><p>left (${left}) > right (${right})</p><p>❌ O elemento <strong>${target.value}</strong> não está no array.</p>`,
      tip: 'Quando left > right, sabemos que o elemento não existe.',
      codeLine: 18
    })
  }

  return stepsArray
}

const generateExplanation = (left, right, mid, midValue, targetValue) => {
  if (midValue === targetValue) {
    return `
      <p>✅ <strong>Encontramos o elemento!</strong></p>
      <p>array[${mid}] = ${midValue} = ${targetValue}</p>
      <p>O elemento foi encontrado no índice <strong>${mid}</strong>.</p>
    `;
  }

  if (midValue < targetValue) {
    return `
      <p>Comparando: array[${mid}] = <strong>${midValue}</strong> com alvo = <strong>${targetValue}</strong></p>
      <p>Como ${midValue} < ${targetValue}, o alvo deve estar <strong>à DIREITA</strong>.</p>
      <p>➡️ Atualizamos: <code>left = mid + 1 = ${mid + 1}</code></p>
    `;
  } else {
    return `
      <p>Comparando: array[${mid}] = <strong>${midValue}</strong> com alvo = <strong>${targetValue}</strong></p>
      <p>Como ${midValue} > ${targetValue}, o alvo deve estar <strong>à ESQUERDA</strong>.</p>
      <p>⬅️ Atualizamos: <code>right = mid - 1 = ${mid - 1}</code></p>
    `;
  }
}

const generateTip = (searchSpace, iteration) => {
  if (iteration === 0) {
    return 'Na primeira iteração, analisamos o elemento do meio do array completo.'
  }
  if (searchSpace <= 2) {
    return 'Com poucos elementos restantes, estamos próximos da resposta!'
  }
  return `A cada iteração, reduzimos o espaço pela metade. Restam ${searchSpace} elementos.`
}

// Player controls
const togglePlay = () => {
  isPlaying.value = !isPlaying.value

  if (isPlaying.value) {
    playInterval.value = setInterval(() => {
      if (currentStep.value < steps.value.length - 1) {
        currentStep.value++
      } else {
        isPlaying.value = false
        clearInterval(playInterval.value)
      }
    }, speed.value)
  } else {
    clearInterval(playInterval.value)
  }
}

const nextStep = () => {
  if (currentStep.value < steps.value.length - 1) {
    currentStep.value++
  }
}

const previousStep = () => {
  if (currentStep.value > 0) {
    currentStep.value--
  }
}

const onTimelineChange = () => {
  if (isPlaying.value) {
    isPlaying.value = false
    clearInterval(playInterval.value)
  }
}

const reset = () => {
  isPlaying.value = false
  initialized.value = false
  currentStep.value = 0
  steps.value = []
  clearInterval(playInterval.value)
}

// Helper functions
const getSpeedLabel = () => {
  if (speed.value <= 400) return '2x'
  if (speed.value <= 800) return '1.5x'
  if (speed.value <= 1200) return '1x'
  return '0.5x'
}

const getStepDescription = () => {
  if (!currentStepData.value) return ''
  if (currentStepData.value.found) return '✓ Encontrado!'
  if (currentStepData.value.comparison === 'notfound') return '❌ Não encontrado'
  return `Comparando... (espaço: ${currentStepData.value.searchSpace})`
}

const getCurrentSearchSpace = () => {
  return currentStepData.value?.searchSpace || 0
}

const getStatus = () => {
  if (!currentStepData.value) return '-'
  if (currentStepData.value.found) return '✓ Encontrado'
  if (currentStepData.value.comparison === 'notfound') return '❌ Não encontrado'
  return '🔍 Buscando...'
}

const getResultClass = () => {
  if (!currentStepData.value) return ''
  if (currentStepData.value.found) return 'success'
  if (currentStepData.value.comparison === 'notfound') return 'failure'
  return ''
}

const getCellClass = (index) => {
  if (!currentStepData.value) return ''

  const classes = []

  if (currentStepData.value.found && index === currentStepData.value.mid) {
    classes.push('found')
  }

  if (index === currentStepData.value.left) classes.push('left')
  if (index === currentStepData.value.right) classes.push('right')
  if (index === currentStepData.value.mid && !currentStepData.value.found) classes.push('mid')

  if (index < currentStepData.value.left || index > currentStepData.value.right) {
    classes.push('eliminated')
  }

  return classes.join(' ')
}

const showPointer = (index, type) => {
  if (!currentStepData.value) return false
  if (type === 'left') return index === currentStepData.value.left
  if (type === 'right') return index === currentStepData.value.right
  if (type === 'mid') return index === currentStepData.value.mid && !currentStepData.value.found
  return false
}

// Watch speed changes
watch(speed, (newSpeed) => {
  if (isPlaying.value) {
    clearInterval(playInterval.value)
    playInterval.value = setInterval(() => {
      if (currentStep.value < steps.value.length - 1) {
        currentStep.value++
      } else {
        isPlaying.value = false
        clearInterval(playInterval.value)
      }
    }, newSpeed)
  }
})

</script>

<style scoped>
.binary-search-enhanced {
  padding: var(--spacing-lg);
  background-color: var(--color-background-alt);
  border-radius: var(--radius-lg);
}

.controls-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: var(--spacing-lg);
  margin-bottom: var(--spacing-lg);
}

.control-section {
  background-color: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}

.control-section h4 {
  margin: 0 0 var(--spacing-md) 0;
  color: var(--color-text-primary);
}

.button-group {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-sm);
}

.input-group {
  display: flex;
  gap: var(--spacing-md);
  align-items: flex-end;
  margin-bottom: var(--spacing-md);
}

.input-group label {
  font-weight: var(--font-weight-medium);
  font-size: var(--text-sm);
}

.form-input {
  padding: var(--spacing-sm);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: var(--text-base);
  width: 100px;
}

.player-controls {
  display: flex;
  gap: var(--spacing-sm);
  align-items: center;
  justify-content: center;
}

.btn-play {
  width: 48px;
  height: 48px;
  font-size: var(--text-2xl);
  padding: 0;
}

.speed-control label {
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
  font-weight: var(--font-weight-medium);
}

.speed-slider {
  flex: 1;
}

.speed-label {
  font-weight: var(--font-weight-bold);
  color: var(--color-primary-600);
  min-width: 40px;
}

.timeline-section {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  margin-bottom: var(--spacing-lg);
  box-shadow: var(--shadow-sm);
}

.timeline-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: var(--spacing-sm);
  font-weight: var(--font-weight-medium);
  color: var(--color-text-secondary);
  font-size: var(--text-sm);
}

.timeline-slider {
  width: 100%;
}

.main-content-grid {
  display: grid;
  grid-template-columns: 1fr 3fr;
  gap: var(--spacing-lg);
  align-items: start;
  margin-bottom: var(--spacing-lg);
}

.status-info {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--spacing-md);
  border-radius: var(--radius-md);
  text-align: center;
  border: 1px solid var(--color-border);
}

.stat-card.success {
  border-color: var(--color-secondary-300);
  background: var(--color-secondary-50);
}

.stat-card.failure {
  border-color: var(--color-error-300);
  background: var(--color-error-50);
}

.stat-label {
  font-size: var(--text-sm);
  color: var(--color-text-secondary);
  margin-bottom: var(--spacing-xs);
  display: block;
}

.stat-value {
  font-size: var(--text-xl);
  font-weight: var(--font-weight-bold);
  color: var(--color-text-primary);
}

.visualization {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}

.array-container {
  display: flex;
  gap: var(--spacing-xs);
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

.left-pointer { color: var(--color-primary-700); background: var(--color-primary-100); }
.right-pointer { color: var(--color-error-700); background: var(--color-error-100); }
.mid-pointer { color: var(--color-accent-700); background: var(--color-accent-100); }

.details-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: var(--spacing-lg);
}

.contextual-explanation {
  background: var(--color-secondary-50);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  border-left: 4px solid var(--color-secondary-500);
}

.explanation-header {
  font-weight: var(--font-weight-semibold);
  margin-bottom: var(--spacing-md);
}

.explanation-content p {
  margin: var(--spacing-sm) 0;
}

.explanation-content code {
  background: var(--color-neutral-200);
  padding: 2px 4px;
  border-radius: var(--radius-sm);
  font-family: var(--font-mono);
}

.explanation-tip {
  margin-top: var(--spacing-md);
  padding: var(--spacing-md);
  background: var(--color-accent-50);
  border-radius: var(--radius-md);
  border-left: 4px solid var(--color-accent-500);
  font-size: var(--text-sm);
}

.code-view {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}

.code-view h4 {
  margin: 0 0 var(--spacing-md) 0;
}

.code-view pre {
  background: var(--color-neutral-900);
  color: var(--color-neutral-100);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  overflow-x: auto;
  font-family: var(--font-mono);
  line-height: 1.6;
}

.code-view code span {
  display: block;
  padding: 0 var(--spacing-sm);
  border-left: 3px solid transparent;
  transition: background-color 0.3s;
}

.code-view code span.highlighted {
  background-color: var(--color-primary-900);
  border-left-color: var(--color-primary-500);
}

@media (max-width: 900px) {
  .main-content-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 600px) {
  .controls-grid, .details-grid {
    grid-template-columns: 1fr;
  }
}
</style>