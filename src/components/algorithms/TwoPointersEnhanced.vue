<template>
  <div class="two-pointers-enhanced">
       <!-- Controles -->
    <div class="controls-grid">
      <div class="control-section quick-tests">
        <h4>🧪 Casos de Teste</h4>
        <div class="button-group">
          <button @click="loadTest1" class="btn btn-sm btn-outline">Encontrado (Início)</button>
          <button @click="loadTest2" class="btn btn-sm btn-outline">Encontrado (Fim)</button>
          <button @click="loadTest3" class="btn btn-sm btn-outline">Não Encontrado</button>
          <button @click="loadTest4" class="btn btn-sm btn-outline">Grande</button>
        </div>
      </div>

      <div class="control-section main-controls">
        <h4>🎮 Controles</h4>
        <div class="input-group">
            <label>Soma alvo:
                <input v-model.number="targetSum" type="number" @change="generateNewSteps" class="form-input" />
            </label>
            <button @click="editArray" class="btn btn-outline">✏️ Editar</button>
        </div>
        <div class="player-controls">
          <button @click="previousStep" :disabled="currentStep === 0" class="btn btn-ghost">⏮</button>
          <button @click="togglePlay" :disabled="steps.length === 0" class="btn btn-secondary btn-play">
            {{ isPlaying ? '⏸' : '▶' }}
          </button>
          <button @click="nextStep" :disabled="currentStep >= steps.length - 1" class="btn btn-ghost">⏭</button>
          <button @click="reset" class="btn btn-error">🔄</button>
        </div>
      </div>

      <div class="control-section speed-control">
        <h4>⚡ Velocidade</h4>
        <input
          v-model.number="speed"
          type="range"
          min="200"
          max="2000"
          step="200"
          class="speed-slider"
        />
        <span class="speed-label">{{ (2200 - speed) / 1000 }}x</span>
      </div>
    </div>

    <!-- Timeline -->
    <div v-if="steps.length > 0" class="timeline-section">
      <div class="timeline-header">
        <span>Passo {{ currentStep + 1 }} de {{ steps.length }}</span>
      </div>
      <input
        v-model.number="currentStep"
        type="range"
        :max="steps.length - 1"
        class="timeline-slider"
      />
    </div>

    <!-- Main Content -->
    <div class="main-content-grid">
        <div class="status-and-explanation">
            <div class="statistics" v-if="currentStepData">
                <div class="stat-card">
                    <div class="stat-label">Iteração</div>
                    <div class="stat-value">{{ currentStepData.iteration }}</div>
                </div>
                <div class="stat-card">
                    <div class="stat-label">Soma Atual</div>
                    <div class="stat-value">{{ currentStepData.currentSum }}</div>
                </div>
                <div class="stat-card" :class="{ success: currentStepData.found, failure: !currentStepData.found && currentStepData.searchSpace < 2 }">
                    <div class="stat-label">Status</div>
                    <div class="stat-value">{{ currentStepData.found ? '✓ Encontrado!' : currentStepData.searchSpace < 2 ? '✗ Não encontrado' : '🔍 Buscando...' }}</div>
                </div>
            </div>
             <div v-if="currentStepData" class="contextual-explanation">
                <div class="explanation-header">💡 O que está acontecendo</div>
                <div class="explanation-content" v-html="currentStepData.explanation"></div>
                <div v-if="currentStepData.tip" class="explanation-tip"><strong>Dica:</strong> {{ currentStepData.tip }}</div>
            </div>
        </div>
        <div class="array-display visualization">
            <h4><span class="enhanced-tab-emoji">👉</span> Array e Ponteiros</h4>
            <div class="array-container">
            <div
                v-for="(value, index) in array"
                :key="index"
                class="array-cell"
                :class="{
                left: currentStepData && index === currentStepData.left,
                right: currentStepData && index === currentStepData.right,
                eliminated: currentStepData && (index < currentStepData.left || index > currentStepData.right),
                found: currentStepData && currentStepData.found && (index === currentStepData.left || index === currentStepData.right)
                }"
            >
                <div class="index">{{ index }}</div>
                <div class="value">{{ value }}</div>
                <div v-if="currentStepData && index === currentStepData.left" class="pointer left-pointer">LEFT</div>
                <div v-if="currentStepData && index === currentStepData.right" class="pointer right-pointer">RIGHT</div>
            </div>
            </div>
        </div>
    </div>

    <!-- Code and Explanation -->
    <div class="details-grid">
        <div class="code-view">
            <h4>💻 Código C++ (Convergência)</h4>
            <pre><code><span v-for="(line, idx) in codeLines" :key="idx" :class="{ highlighted: idx === currentCodeLine }">{{ line }}</span></code></pre>
        </div>
        <div class="pattern-explanation">
            <h3>📚 Padrão de Convergência</h3>
            <p>
                Este padrão utiliza dois ponteiros que começam nas <strong>extremidades opostas</strong> do array
                e se movem em direção ao centro, baseando-se em alguma condição.
            </p>
            <div class="pattern-use-cases">
                <div class="use-case">
                <h4>✅ Quando usar</h4>
                <ul>
                    <li>Array está ordenado</li>
                    <li>Procurar par de elementos com soma específica</li>
                    <li>Encontrar triplas ou pares com propriedades específicas</li>
                </ul>
                </div>
                <div class="use-case">
                <h4>⚡ Vantagens</h4>
                <ul>
                    <li>Reduz complexidade de O(n²) para O(n)</li>
                    <li>Elimina espaço de busca em cada iteração</li>
                </ul>
                </div>
            </div>
        </div>
    </div>

  </div>
</template>

<script setup>
// ... (script content remains the same)
import { ref, computed, watch } from 'vue'

const array = ref([2, 5, 8, 12, 16, 23, 38, 45, 56])
const targetSum = ref(50)
const steps = ref([])
const currentStep = ref(0)
const isPlaying = ref(false)
const speed = ref(1000)
let playInterval = null

const codeLines = [
  'bool findPair(vector<int>& arr, int target) {',
  '  int left = 0;',
  '  int right = arr.size() - 1;',
  '  ',
  '  while (left < right) {',
  '    int sum = arr[left] + arr[right];',
  '    ',
  '    if (sum == target) {',
  '      return true;  // Par encontrado',
  '    }',
  '    else if (sum < target) {',
  '      left++;  // Precisamos de soma maior',
  '    }',
  '    else {',
  '      right--;  // Precisamos de soma menor',
  '    }',
  '  }',
  '  ',
  '  return false;  // Nenhum par encontrado',
  '}'
]

const currentCodeLine = ref(-1)

const currentStepData = computed(() => {
  if (steps.value.length === 0) return null
  return steps.value[currentStep.value]
})

const generateSteps = () => {
  const stepsArray = []
  let left = 0
  let right = array.value.length - 1
  let iteration = 0

  stepsArray.push({
    left,
    right,
    iteration: 0,
    currentSum: array.value[left] + array.value[right],
    searchSpace: right - left + 1,
    found: false,
    explanation: `<strong>Inicialização:</strong><br>- LEFT aponta para o primeiro elemento (índice 0)<br>- RIGHT aponta para o último elemento (índice ${right})<br>- Soma alvo: ${targetSum.value}`,
    tip: 'Em arrays ordenados, podemos usar dois ponteiros para reduzir a complexidade',
    codeLine: 1
  })

  while (left < right) {
    iteration++
    const currentSum = array.value[left] + array.value[right]

    if (currentSum === targetSum.value) {
        stepsArray.push({
            left,
            right,
            iteration,
            currentSum,
            searchSpace: right - left + 1,
            found: true,
            explanation: `<strong>✓ Par Encontrado!</strong><br>- ${array.value[left]} + ${array.value[right]} = ${targetSum.value}<br>- Índices: [${left}, ${right}]`,
            tip: `Complexidade: O(n) - muito melhor que O(n²) da busca de força bruta`,
            codeLine: 8
        })
        return stepsArray
    }

    if (currentSum < targetSum.value) {
      stepsArray.push({
        left,
        right,
        iteration,
        currentSum,
        searchSpace: right - left + 1,
        found: false,
        explanation: `<strong>Movendo LEFT:</strong><br>- Soma atual (${currentSum}) < Soma alvo (${targetSum.value})<br>- Incrementamos LEFT para ${left + 1}`,
        tip: 'Aumentar LEFT aumenta a soma porque o array está ordenado',
        codeLine: 11
      })
      left++
    } else {
      stepsArray.push({
        left,
        right,
        iteration,
        currentSum,
        searchSpace: right - left + 1,
        found: false,
        explanation: `<strong>Movendo RIGHT:</strong><br>- Soma atual (${currentSum}) > Soma alvo (${targetSum.value})<br>- Decrementamos RIGHT para ${right - 1}`,
        tip: 'Diminuir RIGHT diminui a soma porque o array está ordenado',
        codeLine: 14
      })
      right--
    }
  }

  stepsArray.push({
    left,
    right,
    iteration,
    currentSum: 0,
    searchSpace: 0,
    found: false,
    explanation: `<strong>✗ Par Não Encontrado</strong><br>- LEFT e RIGHT se cruzaram.<br>- Exploramos todo o array.`,
    tip: 'Se os ponteiros se cruzam sem encontrar, não existe solução',
    codeLine: 18
  })

  return stepsArray
}

const generateNewSteps = () => {
  currentStep.value = 0
  isPlaying.value = false
  if (playInterval) {
    clearInterval(playInterval)
    playInterval = null
  }
  steps.value = generateSteps()
}

const reset = () => { generateNewSteps() }
const togglePlay = () => {
  isPlaying.value = !isPlaying.value
  if (isPlaying.value) {
    playInterval = setInterval(() => {
      if (currentStep.value < steps.value.length - 1) currentStep.value++
      else { isPlaying.value = false; clearInterval(playInterval); }
    }, speed.value)
  } else { clearInterval(playInterval); }
}
const nextStep = () => { if (currentStep.value < steps.value.length - 1) currentStep.value++; }
const previousStep = () => { if (currentStep.value > 0) currentStep.value--; }
const loadTest1 = () => { array.value = [1, 3, 5, 7, 9, 11]; targetSum.value = 8; generateNewSteps() }
const loadTest2 = () => { array.value = [2, 5, 8, 12, 16, 23, 38, 45, 56]; targetSum.value = 101; generateNewSteps() }
const loadTest3 = () => { array.value = [1, 2, 3, 4, 5]; targetSum.value = 100; generateNewSteps() }
const loadTest4 = () => { array.value = Array.from({length: 20}, (_, i) => i + 1); targetSum.value = 35; generateNewSteps() }

const editArray = () => {
  const input = prompt('Digite o array (separado por vírgulas):', array.value.join(', '))
  if (input) {
    const newArray = input.split(',').map(x => parseInt(x.trim())).filter(x => !isNaN(x))
    if (newArray.length > 0) {
      newArray.sort((a, b) => a - b)
      array.value = newArray
      generateNewSteps()
    }
  }
}

watch(speed, (newSpeed) => {
  if (isPlaying.value) {    clearInterval(playInterval);
    playInterval = setInterval(() => {
      if (currentStep.value < steps.value.length - 1) currentStep.value++
      else { isPlaying.value = false; clearInterval(playInterval); }
    }, newSpeed);
  }
})

watch(currentStepData, (newData) => { if (newData) currentCodeLine.value = newData.codeLine; })
steps.value = generateSteps()
</script>

<style scoped>
/* Main container */
.two-pointers-enhanced {
  padding: var(--spacing-lg);
  background-color: var(--color-background-alt);
  border-radius: var(--radius-lg);
}

/* Layout Grids */
.controls-grid, .details-grid, .main-content-grid {
  display: grid;
  gap: var(--spacing-lg);
  margin-bottom: var(--spacing-lg);
}
.controls-grid { grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); }
.main-content-grid { grid-template-columns: 1fr 2fr; align-items: start; }
.details-grid { grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); }

/* Sections and Panels */
.control-section, .timeline-section, .visualization, .status-and-explanation, .code-view, .pattern-explanation {
  background-color: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}
.control-section h4, .visualization h4, .code-view h4, .pattern-explanation h3 {
  margin: 0 0 var(--spacing-md) 0;
  color: var(--color-text-primary);
}

/* Controls */
.button-group { display: flex; flex-wrap: wrap; gap: var(--spacing-sm); }
.input-group { display: flex; gap: var(--spacing-md); align-items: flex-end; margin-bottom: var(--spacing-md); }
.input-group label { font-weight: var(--font-weight-medium); font-size: var(--text-sm); }
.form-input { padding: var(--spacing-sm); border: 1px solid var(--color-border); border-radius: var(--radius-md); width: 100px; }
.player-controls { display: flex; gap: var(--spacing-sm); align-items: center; justify-content: center; }
.btn-play { width: 48px; height: 48px; font-size: var(--text-2xl); padding: 0; }

/* Speed Control */
.speed-control { display: flex; flex-direction: column; justify-content: center; }
.speed-slider { width: 100%; }
.speed-label { text-align: center; font-weight: var(--font-weight-bold); color: var(--color-primary-600); margin-top: var(--spacing-sm); }

/* Timeline */
.timeline-header { display: flex; justify-content: space-between; margin-bottom: var(--spacing-sm); font-weight: var(--font-weight-medium); color: var(--color-text-secondary); font-size: var(--text-sm); }
.timeline-slider { width: 100%; }

/* Status Info */
.statistics { display: grid; grid-template-columns: repeat(auto-fit, minmax(120px, 1fr)); gap: var(--spacing-md); }
.stat-card { background: var(--color-surface); padding: var(--spacing-md); border-radius: var(--radius-md); text-align: center; border: 1px solid var(--color-border); }
.stat-label { font-size: var(--text-sm); color: var(--color-text-secondary); margin-bottom: var(--spacing-xs); display: block; }
.stat-value { font-size: var(--text-lg); font-weight: var(--font-weight-bold); font-family: var(--font-mono); }
.stat-card.success .stat-value { color: var(--color-secondary-600); }
.stat-card.failure .stat-value { color: var(--color-error-600); }

/* Visualization */
.array-container { display: flex; gap: var(--spacing-xs); overflow-x: auto; padding: calc(var(--spacing-xl) + var(--spacing-sm)) var(--spacing-sm); }
.array-cell { min-width: 60px; height: 80px; border: 1px solid var(--color-border); border-radius: var(--radius-md); display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; background: var(--color-surface); transition: all 0.4s ease; }
.array-cell.eliminated { opacity: 0.3; background: var(--color-neutral-100); }
.array-cell.left { border-color: var(--color-primary-500); background: var(--color-primary-50); transform: scale(1.05); }
.array-cell.right { border-color: var(--color-error-500); background: var(--color-error-50); transform: scale(1.05); }
.array-cell.found { border-color: var(--color-secondary-500); background: var(--color-secondary-100); transform: scale(1.1); animation: found 0.6s ease; }
@keyframes found { 0%, 100% { transform: scale(1.1); } 50% { transform: scale(1.2); } }
.index { position: absolute; top: calc(-1 * var(--spacing-lg)); font-size: var(--text-xs); color: var(--color-text-tertiary); }
.value { font-size: var(--text-lg); font-weight: var(--font-weight-semibold); font-family: var(--font-mono); }
.pointer { position: absolute; bottom: calc(-1 * var(--spacing-xl)); font-size: var(--text-xs); font-weight: var(--font-weight-bold); padding: var(--spacing-xs) var(--spacing-sm); border-radius: var(--radius-sm); }
.left-pointer { color: var(--color-primary-700); background: var(--color-primary-100); }
.right-pointer { color: var(--color-error-700); background: var(--color-error-100); }

/* Explanations */
.contextual-explanation { background: var(--color-secondary-50); padding: var(--spacing-lg); border-radius: var(--radius-md); border-left: 4px solid var(--color-secondary-500); }
.explanation-header { font-weight: var(--font-weight-semibold); margin-bottom: var(--spacing-md); }
.explanation-content p { margin: var(--spacing-sm) 0; }
.explanation-content code { background: var(--color-neutral-200); padding: 2px 4px; border-radius: var(--radius-sm); font-family: var(--font-mono); }
.explanation-tip { margin-top: var(--spacing-md); padding: var(--spacing-md); background: var(--color-accent-50); border-radius: var(--radius-md); border-left: 4px solid var(--color-accent-500); font-size: var(--text-sm); }
.pattern-explanation { background: var(--color-primary-50); border-left: 4px solid var(--color-primary-500); padding: var(--spacing-lg); border-radius: var(--radius-md); }
.pattern-explanation p { line-height: 1.6; }
.pattern-use-cases { display: grid; grid-template-columns: 1fr 1fr; gap: var(--spacing-lg); margin-top: var(--spacing-md); }
.use-case ul { padding-left: var(--spacing-lg); }

/* Code View */
.code-view pre { background: var(--color-neutral-900); color: var(--color-neutral-100); padding: var(--spacing-lg); border-radius: var(--radius-md); overflow-x: auto; font-family: var(--font-mono); line-height: 1.6; }
.code-view code span { display: block; padding: 0 var(--spacing-sm); border-left: 3px solid transparent; transition: background-color 0.3s; }
.code-view code span.highlighted { background-color: var(--color-primary-900); border-left-color: var(--color-primary-500); }

/* Responsive */
@media (max-width: 1200px) {
  .main-content-grid { grid-template-columns: 1fr; }
}
@media (max-width: 600px) {
  .controls-grid, .details-grid, .pattern-use-cases, .statistics {
    grid-template-columns: 1fr;
  }
}
</style>