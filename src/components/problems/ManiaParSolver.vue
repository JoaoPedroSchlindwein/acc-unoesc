<template>
  <div class="solver-container">
    <div class="solver-section">
      <h4>Configuração do Grafo</h4>

      <div class="preset-buttons">
        <button @click="loadPreset('example1')" class="btn btn-sm btn-outline">Exemplo 1 (5)</button>
        <button @click="loadPreset('example2')" class="btn btn-sm btn-outline">Exemplo 2 (-1)</button>
        <button @click="loadPreset('example3')" class="btn btn-sm btn-outline">Exemplo 3 (7)</button>
      </div>

      <div class="graph-input">
        <div class="input-row">
          <label> C (cidades): <input v-model.number="C" type="number" min="2" max="10" class="form-input"/> </label>
          <label> V (estradas): <input v-model.number="V" type="number" min="1" max="20" class="form-input"/> </label>
        </div>

        <div class="edges-input">
          <h5>Estradas (origem, destino, custo):</h5>
          <div v-for="(edge, index) in edges" :key="index" class="edge-input">
            <input v-model.number="edge.u" type="number" placeholder="u" min="1" :max="C" class="form-input"/>
            <input v-model.number="edge.v" type="number" placeholder="v" min="1" :max="C" class="form-input"/>
            <input v-model.number="edge.cost" type="number" placeholder="custo" min="1" class="form-input"/>
            <button @click="removeEdge(index)" class="btn btn-sm btn-error">×</button>
          </div>
          <button @click="addEdge" class="btn btn-sm btn-secondary">+ Adicionar Estrada</button>
        </div>

        <button @click="solve" :disabled="!isValidInput" class="btn btn-primary btn-lg">
          Resolver com Dijkstra
        </button>
        <p v-if="!isValidInput" class="warning">
          ⚠ Verifique se todas as estradas estão preenchidas corretamente.
        </p>
      </div>
    </div>

    <div v-if="solved" class="solver-section solution-section">
      <h4>Solução com Dijkstra + Estado de Paridade</h4>

      <div class="concept-explanation">
        <h5>💡 Conceito: Duplicação de Estados</h5>
        <p>Como o Dijkstra tradicional não rastreia o número de arestas, <strong>duplicamos o espaço de estados</strong>:</p>
        <div class="state-explanation">
          <div class="state-box par"><strong>Estado (v, PAR)</strong><p>Menor custo para chegar em v com nº PAR de pedágios.</p></div>
          <div class="state-box impar"><strong>Estado (v, ÍMPAR)</strong><p>Menor custo para chegar em v com nº ÍMPAR de pedágios.</p></div>
        </div>
      </div>

      <div class="visualization">
        <div class="graph-display">
          <svg :width="graphWidth" :height="graphHeight" class="graph-svg">
            <g>
              <line v-for="(edge, idx) in visualEdges" :key="`edge-${idx}`" :x1="nodePositions[edge.u].x" :y1="nodePositions[edge.u].y" :x2="nodePositions[edge.v].x" :y2="nodePositions[edge.v].y" class="edge"/>
              <text v-for="(edge, idx) in visualEdges" :key="`label-${idx}`" :x="(nodePositions[edge.u].x + nodePositions[edge.v].x) / 2" :y="(nodePositions[edge.u].y + nodePositions[edge.v].y) / 2 - 8" class="edge-label">{{ edge.cost }}</text>
            </g>
            <g>
              <g v-for="city in C" :key="city">
                <circle :cx="nodePositions[city].x" :cy="nodePositions[city].y" r="25" class="node" :class="{ start: city === 1, end: city === C }"/>
                <text :x="nodePositions[city].x" :y="nodePositions[city].y" class="node-label">{{ city }}</text>
              </g>
            </g>
          </svg>
        </div>

        <div class="distances-table">
          <h5>Tabela de Distâncias</h5>
          <table>
            <thead><tr><th>Cidade</th><th class="par-col">PAR</th><th class="impar-col">ÍMPAR</th></tr></thead>
            <tbody>
              <tr v-for="city in C" :key="city">
                <td><strong>{{ city }}</strong></td>
                <td class="par-col">{{ dist[city][0] === Infinity ? '∞' : dist[city][0] }}</td>
                <td class="impar-col">{{ dist[city][1] === Infinity ? '∞' : dist[city][1] }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div class="result" :class="{ impossible: result === -1 }">
        <div class="result-value"><strong>Resposta:</strong> {{ result === -1 ? 'Impossível (-1)' : `${result} reais` }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
// ... (script content remains the same)
import { ref, computed } from 'vue'

const C = ref(4)
const V = ref(4)
const edges = ref([
  { u: 1, v: 2, cost: 3 },
  { u: 2, v: 4, cost: 2 },
  { u: 1, v: 3, cost: 4 },
  { u: 3, v: 4, cost: 1 }
])

const solved = ref(false)
const dist = ref({})
const result = ref(0)
const path = ref([])

const graphWidth = 600
const graphHeight = 400

const Infinity = Number.POSITIVE_INFINITY

const isValidInput = computed(() => {
  return edges.value.length > 0 &&
         edges.value.every(e => e.u && e.v && e.cost && e.u <= C.value && e.v <= C.value)
})

const nodePositions = computed(() => {
  const positions = {}
  const centerX = graphWidth / 2
  const centerY = graphHeight / 2
  const radius = Math.min(centerX, centerY) - 40

  for (let i = 1; i <= C.value; i++) {
    const angle = (2 * Math.PI * (i - 1)) / C.value - Math.PI / 2
    positions[i] = {
      x: centerX + radius * Math.cos(angle),
      y: centerY + radius * Math.sin(angle)
    }
  }

  return positions
})

const visualEdges = computed(() => {
  return edges.value.map(e => ({
    u: e.u,
    v: e.v,
    cost: e.cost,
    inPath: false
  }))
})

const addEdge = () => { edges.value.push({ u: null, v: null, cost: null }) }
const removeEdge = (index) => { edges.value.splice(index, 1) }

const loadPreset = (preset) => {
  if (preset === 'example1') {
    C.value = 4; V.value = 4;
    edges.value = [
      { u: 1, v: 2, cost: 3 }, { u: 2, v: 4, cost: 2 },
      { u: 1, v: 3, cost: 4 }, { u: 3, v: 4, cost: 1 }
    ]
  } else if (preset === 'example2') {
    C.value = 2; V.value = 0; edges.value = []
  } else if (preset === 'example3') {
    C.value = 5; V.value = 5;
    edges.value = [
      { u: 1, v: 2, cost: 2 }, { u: 2, v: 3, cost: 1 },
      { u: 3, v: 4, cost: 1 }, { u: 4, v: 5, cost: 3 },
      { u: 1, v: 5, cost: 10 }
    ]
  }
  solved.value = false
}

const solve = () => {
  const adj = {}; for (let i = 1; i <= C.value; i++) adj[i] = [];
  for (const edge of edges.value) {
    adj[edge.u].push({ v: edge.v, cost: edge.cost });
    adj[edge.v].push({ v: edge.u, cost: edge.cost });
  }
  const distances = {}; for (let i = 1; i <= C.value; i++) distances[i] = [Infinity, Infinity];
  const pq = [];
  distances[1][0] = 0;
  pq.push({ cost: 0, city: 1, parity: 0 });

  while (pq.length > 0) {
    pq.sort((a, b) => a.cost - b.cost);
    const { cost: d_u, city: u, parity: parity_u } = pq.shift();
    if (d_u > distances[u][parity_u]) continue;
    for (const { v, cost: w } of adj[u]) {
      const new_parity = 1 - parity_u;
      const new_cost = distances[u][parity_u] + w;
      if (new_cost < distances[v][new_parity]) {
        distances[v][new_parity] = new_cost;
        pq.push({ cost: new_cost, city: v, parity: new_parity });
      }
    }
  }
  dist.value = distances;
  const finalDist = distances[C.value][0];
  result.value = finalDist === Infinity ? -1 : finalDist;
  solved.value = true;
}

</script>

<style scoped>
.solver-container { display: flex; flex-direction: column; gap: var(--spacing-lg); }
.solver-section { background: var(--color-surface); padding: var(--spacing-lg); border-radius: var(--radius-lg); border: 1px solid var(--color-border); }
.solver-section h4 { margin: 0 0 var(--spacing-md) 0; }

.preset-buttons { display: flex; gap: var(--spacing-sm); margin-bottom: var(--spacing-lg); flex-wrap: wrap; }

.graph-input { display: flex; flex-direction: column; gap: var(--spacing-lg); }
.input-row { display: flex; gap: var(--spacing-lg); flex-wrap: wrap; }
.input-row label { display: flex; flex-direction: column; gap: var(--spacing-sm); font-weight: var(--font-weight-medium); font-size: var(--text-sm); }
.form-input { width: 100px; }

.edges-input { background: var(--color-background-alt); padding: var(--spacing-md); border-radius: var(--radius-md); }
.edges-input h5 { margin-bottom: var(--spacing-md); }
.edge-input { display: flex; gap: var(--spacing-sm); margin-bottom: var(--spacing-sm); align-items: center; }
.edge-input .form-input { width: 80px; }

.warning { color: var(--color-error-600); margin-top: var(--spacing-md); font-weight: var(--font-weight-medium); }

.concept-explanation { background: var(--color-primary-50); padding: var(--spacing-lg); border-radius: var(--radius-md); border-left: 4px solid var(--color-primary-500); }
.state-explanation { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: var(--spacing-md); margin: var(--spacing-md) 0; }
.state-box { padding: var(--spacing-md); border-radius: var(--radius-md); text-align: center; }
.state-box.par { background: var(--color-secondary-100); border: 1px solid var(--color-secondary-300); }
.state-box.impar { background: var(--color-accent-100); border: 1px solid var(--color-accent-300); }

.visualization { display: grid; grid-template-columns: 2fr 1fr; gap: var(--spacing-lg); align-items: center; }
.graph-svg { display: block; margin: 0 auto; background: var(--color-surface); border-radius: var(--radius-md); border: 1px solid var(--color-border); }
.edge { stroke: var(--color-border-strong); stroke-width: 2; }
.edge-label { font-size: var(--text-sm); font-weight: var(--font-weight-semibold); fill: var(--color-text-secondary); }
.node { fill: var(--color-surface); stroke: var(--color-neutral-700); stroke-width: 2; }
.node.start { fill: var(--color-secondary-100); stroke: var(--color-secondary-600); }
.node.end { fill: var(--color-primary-100); stroke: var(--color-primary-600); }
.node-label { font-size: var(--text-base); font-weight: var(--font-weight-bold); fill: var(--color-text-primary); pointer-events: none; }

.distances-table table { width: 100%; border-collapse: collapse; background: var(--color-surface); border-radius: var(--radius-md); overflow: hidden; box-shadow: var(--shadow-sm); }
.distances-table th, .distances-table td { padding: var(--spacing-sm); text-align: center; border: 1px solid var(--color-border); }
.distances-table th { background: var(--color-neutral-800); color: var(--color-text-inverse); }
.par-col { background: var(--color-secondary-50); }
.impar-col { background: var(--color-accent-50); }

.result { background: linear-gradient(135deg, var(--color-primary-600), var(--color-primary-700)); padding: var(--spacing-xl); border-radius: var(--radius-md); text-align: center; color: var(--color-text-inverse); }
.result.impossible { background: linear-gradient(135deg, var(--color-error-600), var(--color-error-700)); }
.result-value { font-size: var(--text-xl); }

@media (max-width: 900px) {
  .visualization { grid-template-columns: 1fr; }
}
</style>