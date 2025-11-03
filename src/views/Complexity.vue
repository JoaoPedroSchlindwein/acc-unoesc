<template>
  <div>
    <h1>Complexidade de Algoritmos</h1>
    <p class="intro">
      Compare visualmente como o número de operações (eixo Y) cresce com o tamanho da entrada N (eixo X). Use o seletor e alterne as escalas para explorar o comportamento de cada complexidade.
    </p>

    <div class="section">
        <h2>Visualização de Escala de Crescimento</h2>
        <p>Passe o mouse sobre a legenda para destacar uma curva. Use o seletor de N para ver os valores exatos em cada ponto.</p>
      
       <div class="control-section">
        <label for="n-slider">Selecione o valor de N:</label>
        <div class="slider-group">
            <input id="n-slider" v-model.number="interactiveN" type="range" :min="nMin" :max="nMax" step="1" class="n-slider"/>
            <span class="n-value">{{ interactiveN }}</span>
        </div>
        <div class="button-group">
            <button class="btn btn-sm" :class="{ 'btn-primary': scaleType === 'linear', 'btn-outline': scaleType !== 'linear' }" @click="scaleType = 'linear'">Escala Linear</button>
            <button class="btn btn-sm" :class="{ 'btn-primary': scaleType === 'log', 'btn-outline': scaleType !== 'log' }" @click="scaleType = 'log'">Escala Log</button>
        </div>
      </div>

      <div class="chart-wrapper">
        <div class="chart-container">
            <LineChart v-if="chartData" :chartData="chartData" :chartOptions="chartOptions" />
            <div v-if="highlightedComplexity" class="chart-tooltip">
                <h4>{{ highlightedComplexity }}</h4>
                <p>{{ descriptions[highlightedComplexity] }}</p>
            </div>
        </div>
        <div class="outputs-and-legend">
            <div class="outputs-container">
                <h4>Operações para N = {{ interactiveN }}</h4>
                <div class="output-list">
                    <div v-for="(ops, key) in operationCounts" :key="`ops-${key}`" class="output-item">
                        <span class="output-key">{{ key }}:</span>
                        <span class="output-value">{{ ops }}</span>
                    </div>
                </div>
            </div>
            <div class="legend">
                <div v-for="(dataset, index) in chartData.datasets" :key="`legend-${index}`" 
                     class="legend-item" 
                     :class="{ 'highlighted': highlightedComplexity === dataset.label }"
                     @mouseover="highlightedComplexity = dataset.label" 
                     @mouseleave="highlightedComplexity = null">
                    <div class="legend-color" :style="{ backgroundColor: dataset.borderColor }"></div>
                    <span>{{ dataset.label }}</span>
                </div>
            </div>
        </div>
      </div>
    </div>

    <div class="section">
      <h2>Guia de Decisão Rápido</h2>
      <div class="decision-guide">
        <div class="guide-item"><strong>n ≤ 20</strong> → <code>O(n!)</code>, <code>O(2^n)</code> podem ser viáveis.</div>
        <div class="guide-item"><strong>n ≤ 100</strong> → <code>O(n³)</code> ou <code>O(n⁴)</code> podem passar.</div>
        <div class="guide-item"><strong>n ≤ 5.000</strong> → <code>O(n²)</code> é geralmente aceitável.</div>
        <div class="guide-item"><strong>n ≤ 10⁶</strong> → Uma solução <code>O(n log n)</code> é necessária.</div>
        <div class="guide-item"><strong>n > 10⁷</strong> → Requer <code>O(n)</code>, <code>O(log n)</code> ou <code>O(1)</code>.</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import LineChart from '../components/common/LineChart.vue';

const highlightedComplexity = ref(null);
const interactiveN = ref(20);
const scaleType = ref('linear'); // 'linear' or 'log'

const nMin = 1;
const nMax = computed(() => (scaleType.value === 'linear' ? 30 : 100));
const opsMaxLinear = 30000;

watch(nMax, (newMax) => {
  if (interactiveN.value > newMax) {
    interactiveN.value = newMax;
  }
});

const descriptions = {
  'O(log n)': 'Extremamente eficiente. O tempo de execução cresce muito lentamente. Usado em busca binária.',
  'O(n)': 'Eficiente e comum. O tempo de execução cresce linearmente com a entrada. Usado para percorrer um array.',
  'O(n log n)': 'Muito eficiente para algoritmos baseados em ordenação (quicksort, mergesort).',
  'O(n²)': 'Aceitável para entradas pequenas (N < 5000). Comum em laços aninhados.',
  'O(n³)': 'Lento. Geralmente envolve três laços aninhados. Viável apenas para N muito pequeno (N < 500).',
  'O(2^n)': 'Crescimento exponencial. Viável apenas para N muito pequeno (N < 25). Comum em problemas de subconjuntos.',
  'O(n!)': 'Extremamente lento e inviável para quase todas as entradas (N > 15). Comum em problemas de permutação por força bruta.',
};

const factorial = (n) => {
    if (n > 20) return Infinity; 
    if (n < 0) return 0;
    if (n === 0 || n === 1) return 1;
    let result = 1;
    for (let i = 2; i <= n; i++) {
      result *= i;
    }
    return result;
};

const functions = {
  'O(log n)': n => (n > 0 ? Math.log2(n) : 0),
  'O(n)': n => n,
  'O(n log n)': n => (n > 0 ? n * Math.log2(n) : 0),
  'O(n²)': n => n * n,
  'O(n³)': n => n * n * n,
  'O(2^n)': n => Math.pow(2, n),
  'O(n!)': n => factorial(n),
};

const chartData = computed(() => {
    const labels = Array.from({ length: interactiveN.value }, (_, i) => i + 1);
    const datasets = Object.keys(functions).map(key => {
        const colorMap = {
            'O(log n)': 'hsl(210, 100%, 56%)',
            'O(n)': 'hsl(155, 100%, 42%)',
            'O(n log n)': 'hsl(185, 100%, 42%)',
            'O(n²)': 'hsl(45, 100%, 50%)',
            'O(n³)': 'hsl(15, 100%, 50%)',
            'O(2^n)': 'hsl(0, 100%, 50%)',
            'O(n!)': 'hsl(300, 100%, 50%)',
        };
        
        const isHighlighted = highlightedComplexity.value === key;

        return {
            label: key,
            data: labels.map(n => {
                const ops = functions[key](n);
                if (scaleType.value === 'linear' && ops > opsMaxLinear) return opsMaxLinear;;
                if (scaleType.value === 'log' && (ops <= 0 || !isFinite(ops))) return null;
                return ops;
            }),
            borderColor: colorMap[key],
            borderWidth: isHighlighted ? 4 : 2.5,
            pointRadius: isHighlighted ? 3 : 0,
            backgroundColor: colorMap[key] + '33', 
            tension: 0.1,
            spanGaps: false,
            fill: false,
        };
    });

    return { labels, datasets };
});

const chartOptions = computed(() => ({
    responsive: true,
    maintainAspectRatio: false,
    interaction: {
        intersect: false,
        mode: 'index',
    },
    onHover: (event, chartElements, chart) => {
        if (chartElements.length > 0) {
            const element = chartElements[0];
            const datasetLabel = chart.data.datasets[element.datasetIndex].label;
            highlightedComplexity.value = datasetLabel;
        } else {
            highlightedComplexity.value = null;
        }
    },
    scales: {
        x: {
            title: { display: true, text: `Tamanho da Entrada (N)` },
            max: interactiveN.value,
        },
}}));

const operationCounts = computed(() => {
    const n = interactiveN.value;
    const counts = {};
    for (const key in functions) {
        const raw_ops = functions[key](n);
        counts[key] = raw_ops > 1000000 ? raw_ops.toExponential(2) : Math.round(raw_ops).toLocaleString();
    }
    return counts;
});

</script>

<style scoped>
/* Styles from previous correct implementation are preserved here */
.intro { font-size: var(--text-lg); color: var(--color-text-secondary); margin-bottom: var(--spacing-2xl); max-width: 800px; }
.section { background: var(--color-surface); padding: var(--spacing-xl); border-radius: var(--radius-lg); margin-bottom: var(--spacing-2xl); box-shadow: var(--shadow-base); }
.section h2 { margin-bottom: var(--spacing-lg); border-bottom: 1px solid var(--color-border); padding-bottom: var(--spacing-md); }

.chart-controls { display: grid; grid-template-columns: 2fr 1fr; gap: var(--spacing-lg); margin-bottom: var(--spacing-lg); }
.control-section { background-color: var(--color-background-alt); padding: var(--spacing-md); border-radius: var(--radius-md); }
.control-section label { display: block; font-weight: var(--font-weight-medium); margin-bottom: var(--spacing-sm); font-size: var(--text-sm); }
.slider-group { display: flex; align-items: center; gap: var(--spacing-md); }
.button-group { display: flex; gap: var(--spacing-sm); }

.chart-wrapper { display: grid; grid-template-columns: 3fr 1fr; gap: var(--spacing-xl); min-height: 600px;}
.chart-container { position: relative; }

.outputs-and-legend { display: flex; flex-direction: column; gap: var(--spacing-lg); }

.legend { display: flex; flex-direction: column; gap: var(--spacing-sm); background-color: var(--color-background-alt); padding: var(--spacing-md); border-radius: var(--radius-md); border: 1px solid var(--color-border); }
.legend-item { display: flex; align-items: center; gap: var(--spacing-sm); cursor: pointer; font-family: var(--font-mono); padding: var(--spacing-xs); border-radius: var(--radius-sm); transition: background-color 0.2s; }
.legend-item.highlighted { background-color: var(--color-neutral-200); font-weight: var(--font-weight-semibold); }
.legend-color { width: 20px; height: 6px; border-radius: 3px; }

.outputs-container { background-color: var(--color-background-alt); padding: var(--spacing-lg); border-radius: var(--radius-md); border: 1px solid var(--color-border); }
.outputs-container h4 { margin: 0 0 var(--spacing-md) 0; }
.output-list { display: flex; flex-direction: column; gap: var(--spacing-sm); font-family: var(--font-mono); font-size: var(--text-sm); }
.output-item { display: flex; justify-content: space-between; }
.output-key { font-weight: var(--font-weight-medium); color: var(--color-text-secondary); }
.output-value { font-weight: var(--font-weight-bold); }

.chart-tooltip { position: absolute; top: 10px; left: 10px; background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(5px); padding: var(--spacing-md); border-radius: var(--radius-md); box-shadow: var(--shadow-lg); border: 1px solid var(--color-border); max-width: 250px; z-index: 10; pointer-events: none; }
.chart-tooltip h4 { margin: 0 0 var(--spacing-sm) 0; }
.chart-tooltip p { margin: 0; font-size: var(--text-sm); }

.n-slider { flex-grow: 1; }
.n-value { font-family: var(--font-mono); font-weight: var(--font-weight-bold); font-size: var(--text-lg); background-color: var(--color-surface); padding: var(--spacing-sm) var(--spacing-md); border-radius: var(--radius-md); border: 1px solid var(--color-border); min-width: 60px; text-align: center; }

.decision-guide { display: grid; gap: var(--spacing-md); }
.guide-item { background: var(--color-secondary-50); padding: var(--spacing-lg); border-left: 4px solid var(--color-secondary-500); border-radius: var(--radius-sm); font-family: var(--font-mono); }

@media (max-width: 900px) {
    .chart-wrapper, .chart-controls {
        grid-template-columns: 1fr;
    }
}
</style>