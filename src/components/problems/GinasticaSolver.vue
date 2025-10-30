<template>
  <div class="solver-container">
    <div class="solver-section">
      <h4>Entrada</h4>
      <div class="inputs">
        <label>
          T (minutos):
          <input v-model.number="T" type="number" min="1" max="10" class="form-input" />
        </label>
        <label>
          M (nível mínimo):
          <input v-model.number="M" type="number" min="1" max="20" class="form-input" />
        </label>
        <label>
          N (nível máximo):
          <input v-model.number="N" type="number" min="1" max="20" class="form-input" />
        </label>
        <button @click="solve" :disabled="M >= N" class="btn btn-primary">
          Resolver
        </button>
      </div>
      <p v-if="M >= N" class="warning">⚠ M deve ser menor que N</p>
    </div>

    <div v-if="solved" class="solver-section solution-section">
      <h4>Solução com Programação Dinâmica</h4>

      <div class="explanation">
        <h5>Conceito:</h5>
        <p>
          Usamos um vetor <code>dp</code> onde <code>dp[i]</code> conta quantas sequências terminam no nível <code>M+i</code>.
        </p>
        <p>
          <strong>Regras de transição:</strong>
        </p>
        <ul>
          <li>Se o nível é M, só pode vir de M+1 (descer).</li>
          <li>Se o nível é N, só pode vir de N-1 (subir).</li>
          <li>Para outros níveis, pode vir de d-1 ou d+1.</li>
        </ul>
      </div>

      <div class="visualization">
        <h5>Evolução do Vetor DP:</h5>
        <div v-for="(state, minute) in dpStates" :key="minute" class="dp-state">
          <div class="minute-label">Minuto {{ minute + 1 }}</div>
          <div class="dp-array">
            <div
              v-for="(value, index) in state"
              :key="index"
              class="dp-cell"
              :class="{ active: value > 0 }"
            >
              <div class="level-label">{{ M + index }}</div>
              <div class="dp-value">{{ value }}</div>
            </div>
          </div>
          <div v-if="minute < dpStates.length - 1" class="arrow">↓</div>
        </div>
      </div>

      <div class="result">
        <div class="result-value">
          <strong>Resposta:</strong> {{ result }} sequências válidas
        </div>
        <div class="result-formula">
          Soma: {{ dpStates[dpStates.length - 1].join(' + ') }} = {{ result }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const T = ref(3)
const M = ref(2)
const N = ref(5)
const solved = ref(false)

const MOD = 1000000007

const dpStates = computed(() => {
  if (!solved.value || M.value >= N.value) return []

  const size = N.value - M.value + 1
  const states = []

  let dp = Array(size).fill(1)
  states.push([...dp])

  for (let t = 2; t <= T.value; t++) {
    const dp_novo = Array(size).fill(0)

    for (let i = 0; i < size; i++) {
      const d = M.value + i
      if (d === M.value) {
        dp_novo[i] = dp[i + 1]
      } else if (d === N.value) {
        dp_novo[i] = dp[i - 1]
      } else {
        dp_novo[i] = (dp[i - 1] + dp[i + 1]) % MOD
      }
    }
    dp = [...dp_novo]
    states.push([...dp])
  }
  return states
})

const result = computed(() => {
  if (dpStates.value.length === 0) return 0
  const lastState = dpStates.value[dpStates.value.length - 1]
  return lastState.reduce((sum, val) => (sum + val) % MOD, 0)
})

const solve = () => {
  if (M.value < N.value) {
    solved.value = true
  }
}
</script>

<style scoped>
.solver-container {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-lg);
}

.solver-section {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
}

.solver-section h4 {
  margin: 0 0 var(--spacing-md) 0;
}

.inputs {
  display: flex;
  gap: var(--spacing-md);
  align-items: flex-end;
  flex-wrap: wrap;
}

.inputs label {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-sm);
  font-weight: var(--font-weight-medium);
  font-size: var(--text-sm);
}

.form-input {
  padding: var(--spacing-sm) var(--spacing-md);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: var(--text-base);
  width: 120px;
}

.warning {
  color: var(--color-error-600);
  margin-top: var(--spacing-md);
  font-weight: var(--font-weight-medium);
}

.explanation {
  background: var(--color-background-alt);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  margin-bottom: var(--spacing-lg);
  border-left: 4px solid var(--color-primary-400);
}

.explanation ul {
  padding-left: var(--spacing-lg);
}

.visualization {
  background: var(--color-background-alt);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  margin-bottom: var(--spacing-lg);
}

.dp-state {
  margin: var(--spacing-lg) 0;
}

.minute-label {
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-primary);
  margin-bottom: var(--spacing-md);
}

.dp-array {
  display: flex;
  gap: var(--spacing-sm);
  justify-content: center;
  flex-wrap: wrap;
}

.dp-cell {
  min-width: 60px;
  padding: var(--spacing-sm);
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  text-align: center;
  transition: all var(--transition-base);
}

.dp-cell.active {
  border-color: var(--color-secondary-500);
  background: var(--color-secondary-50);
}

.level-label {
  font-size: var(--text-xs);
  color: var(--color-text-tertiary);
  margin-bottom: var(--spacing-xs);
}

.dp-value {
  font-size: var(--text-xl);
  font-weight: var(--font-weight-bold);
  color: var(--color-text-primary);
  font-family: var(--font-mono);
}

.arrow {
  text-align: center;
  font-size: var(--text-2xl);
  color: var(--color-secondary-500);
  margin: var(--spacing-sm) 0;
}

.result {
  background: linear-gradient(135deg, var(--color-primary-600), var(--color-primary-700));
  padding: var(--spacing-xl);
  border-radius: var(--radius-md);
  margin: var(--spacing-lg) 0;
  text-align: center;
  color: var(--color-text-inverse);
}

.result-value {
  font-size: var(--text-xl);
  margin-bottom: var(--spacing-sm);
}

.result-formula {
  font-family: var(--font-mono);
  opacity: 0.9;
}

code {
  background: var(--color-neutral-200);
  padding: 2px 6px;
  border-radius: var(--radius-sm);
  font-family: var(--font-mono);
}
</style>