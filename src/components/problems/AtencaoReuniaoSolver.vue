<template>
  <div class="solver-container">
    <div class="solver-section">
      <h4>Entrada</h4>
      <div class="inputs">
        <label>
          N (diretores):
          <input v-model.number="N" type="number" min="1" max="100" class="form-input" />
        </label>
        <label>
          K (duração máxima):
          <input v-model.number="K" type="number" min="1" max="1000" class="form-input" />
        </label>
        <button @click="solve" class="btn btn-primary">Resolver</button>
      </div>
    </div>

    <div v-if="solved" class="solver-section solution-section">
      <h4>Solução Passo a Passo</h4>

      <div class="explanation-group">
        <h5>1. Modelagem Matemática</h5>
        <div class="formula">
          <p>Tempo Discursos: <code>N × T</code></p>
          <p>Nº Intervalos: <code>N - 1</code></p>
          <p>Tempo Intervalos: <code>(N - 1) × 1</code></p>
          <p><strong>Restrição:</strong> <code>N × T + (N - 1) ≤ K</code></p>
        </div>
      </div>

      <div class="explanation-group">
        <h5>2. Resolvendo para T (Tempo)</h5>
        <div class="steps">
          <div class="step">N × T ≤ K - (N - 1)</div>
          <div class="step">N × T ≤ K - N + 1</div>
          <div class="step">T ≤ (K - N + 1) / N</div>
          <div class="step final">T = ⌊({{ K }} - {{ N }} + 1) / {{ N }}⌋</div>
        </div>
      </div>

      <div class="result">
        <div class="result-value">
          <strong>Resposta:</strong> {{ tempoMaximo }} minutos
        </div>
      </div>

      <div class="explanation-group">
        <h5>3. Verificação</h5>
        <div class="verification">
            <p>Tempo de discursos: <code>{{ N }} × {{ tempoMaximo }} = {{ N * tempoMaximo }} min</code></p>
            <p>Tempo de intervalos: <code>{{ N - 1 }} min</code></p>
            <p><strong>Tempo total:</strong> <code>{{ N * tempoMaximo + (N - 1) }} min</code></p>
            <div class="validation" :class="{ valid: N * tempoMaximo + (N - 1) <= K }">
              <strong>Verificação:</strong> 
              <code>{{ N * tempoMaximo + (N - 1) }} ≤ {{ K }}</code>
              <span v-if="N * tempoMaximo + (N - 1) <= K">✓ Válido</span>
              <span v-else>✗ Inválido</span>
            </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const N = ref(7)
const K = ref(120)
const solved = ref(false)

const tempoMaximo = computed(() => {
  if (N.value > 0) {
    return Math.floor((K.value - N.value + 1) / N.value)
  }
  return 0;
})

const solve = () => {
  solved.value = true
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
  width: 150px;
}

.explanation-group {
    margin-top: var(--spacing-lg);
}

.explanation-group h5 {
    margin-bottom: var(--spacing-sm);
}

.formula, .steps, .verification {
  background: var(--color-background-alt);
  padding: var(--spacing-md);
  border-radius: var(--radius-md);
  border: 1px solid var(--color-border);
  font-size: var(--text-sm);
}

.formula p, .steps .step, .verification p {
  margin: var(--spacing-sm) 0;
  font-family: var(--font-mono);
}

.steps .step {
  padding: var(--spacing-sm);
  border-left: 3px solid var(--color-border);
}

.steps .final {
  border-left-color: var(--color-secondary-500);
  background: var(--color-secondary-50);
  font-weight: var(--font-weight-semibold);
}

.result {
  background: linear-gradient(135deg, var(--color-primary-600), var(--color-primary-700));
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  margin: var(--spacing-lg) 0;
  text-align: center;
}

.result-value {
  color: var(--color-text-inverse);
  font-size: var(--text-xl);
  font-weight: var(--font-weight-bold);
}

.validation {
    margin-top: var(--spacing-md);
    padding: var(--spacing-sm);
    border-radius: var(--radius-sm);
    font-weight: var(--font-weight-semibold);
}

.validation.valid {
    background: var(--color-secondary-100);
    color: var(--color-secondary-800);
}

.validation.invalid {
    background: var(--color-error-100);
    color: var(--color-error-700);
}

code {
  background: var(--color-neutral-200);
  padding: 2px 6px;
  border-radius: var(--radius-sm);
  font-family: var(--font-mono);
}
</style>