<template>
  <div class="bfs-enhanced">
    <!-- Controles -->
    <div class="controls-grid">
      <div class="control-section quick-tests">
        <h4>🧪 Casos de Teste</h4>
        <div class="button-group">
          <button @click="loadSmallGraph" class="btn btn-sm btn-outline">Pequeno</button>
          <button @click="loadMediumGraph" class="btn btn-sm btn-outline">Médio</button>
          <button @click="loadLargeGraph" class="btn btn-sm btn-outline">Grande</button>
        </div>
      </div>

      <div class="control-section main-controls">
        <h4>🎮 Controles</h4>
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
        <div v-if="currentStepData" class="status-info">
            <div class="stat-card">
                <div class="stat-label">Nós Visitados</div>
                <div class="stat-value">{{ currentStepData.visited.size }}</div>
            </div>
            <div class="stat-card">
                <div class="stat-label">Nó Atual</div>
                <div class="stat-value">{{ currentStepData.currentNode !== null ? currentStepData.currentNode : '-' }}</div>
            </div>
            <div class="stat-card">
                <div class="stat-label">Fila</div>
                <div class="stat-value queue-display">{{ currentStepData.queue.length > 0 ? '[' + currentStepData.queue.join(', ') + ']' : '[]' }}</div>
            </div>
        </div>

        <div v-if="currentStepData" class="contextual-explanation">
          <div class="explanation-header">💡 O que está acontecendo</div>
          <div class="explanation-content" v-html="currentStepData.explanation"></div>
          <div v-if="currentStepData.tip" class="explanation-tip"><strong>Dica:</strong> {{ currentStepData.tip }}</div>
        </div>
      </div>

      <div class="visualization">
        <div class="graph-container">
            <svg width="700" height="500" class="graph-svg">
            <!-- Edges -->
            <g>
                <line
                v-for="edge in currentEdges"
                :key="`${edge.from}-${edge.to}`"
                :x1="currentNodes[edge.from].x"
                :y1="currentNodes[edge.from].y"
                :x2="currentNodes[edge.to].x"
                :y2="currentNodes[edge.to].y"
                class="edge"
                :class="{
                    explored: currentStepData && isEdgeExplored(edge.from, edge.to),
                    active: currentStepData && isEdgeActive(edge.from, edge.to)
                }"
                />
            </g>

            <!-- Nodes -->
            <g>
                <g v-for="(node, id) in currentNodes" :key="id">
                <circle
                    :cx="node.x"
                    :cy="node.y"
                    :r="30"
                    class="node"
                    :class="{
                    visited: currentStepData && currentStepData.visited.has(parseInt(id)),
                    current: currentStepData && currentStepData.currentNode === parseInt(id),
                    inQueue: currentStepData && currentStepData.queue.includes(parseInt(id)),
                    start: parseInt(id) === 0
                    }"
                />
                <text
                    :x="node.x"
                    :y="node.y"
                    text-anchor="middle"
                    dominant-baseline="middle"
                    class="node-label"
                >
                    {{ id }}
                </text>
                <text
                    v-if="currentStepData && currentStepData.distance[id] !== undefined && currentStepData.distance[id] !== Infinity"
                    :x="node.x"
                    :y="node.y - 45"
                    text-anchor="middle"
                    class="distance-label"
                >
                    d={{ currentStepData.distance[id] }}
                </text>
                </g>
            </g>
            </svg>

            <div class="legend">
                <div class="legend-item"><div class="legend-box start"></div><span>Início</span></div>
                <div class="legend-item"><div class="legend-box unvisited"></div><span>Não Visitado</span></div>
                <div class="legend-item"><div class="legend-box in-queue"></div><span>Na Fila</span></div>
                <div class="legend-item"><div class="legend-box current-node"></div><span>Processando</span></div>
                <div class="legend-item"><div class="legend-box visited-node"></div><span>Visitado</span></div>
            </div>
        </div>
      </div>
    </div>

    <div class="code-view">
      <h4>📝 Código C++ (linha atual destacada)</h4>
      <pre><code><span v-for="(line, idx) in codeLines" :key="idx" :class="{ highlighted: idx === currentCodeLine }">{{ line }}</span></code></pre>
    </div>

  </div>
</template>

<script setup>
// ... (script content remains the same)
import { ref, computed, watch } from 'vue'

// Graph configurations
const graphConfigs = {
  small: {
    nodes: {
      0: { x: 200, y: 100 },
      1: { x: 100, y: 250 },
      2: { x: 300, y: 250 },
      3: { x: 200, y: 350 }
    },
    edges: [
      { from: 0, to: 1 },
      { from: 0, to: 2 },
      { from: 1, to: 3 },
      { from: 2, to: 3 }
    ],
    adjacency: {
      0: [1, 2],
      1: [0, 3],
      2: [0, 3],
      3: [1, 2]
    }
  },
  medium: {
    nodes: {
      0: { x: 350, y: 80 },
      1: { x: 200, y: 180 },
      2: { x: 500, y: 180 },
      3: { x: 120, y: 320 },
      4: { x: 280, y: 320 },
      5: { x: 420, y: 320 },
      6: { x: 560, y: 320 }
    },
    edges: [
      { from: 0, to: 1 },
      { from: 0, to: 2 },
      { from: 1, to: 3 },
      { from: 1, to: 4 },
      { from: 2, to: 5 },
      { from: 2, to: 6 },
      { from: 3, to: 4 }
    ],
    adjacency: {
      0: [1, 2],
      1: [0, 3, 4],
      2: [0, 5, 6],
      3: [1, 4],
      4: [1, 3],
      5: [2],
      6: [2]
    }
  },
  large: {
    nodes: {
      0: { x: 350, y: 50 },
      1: { x: 200, y: 150 },
      2: { x: 500, y: 150 },
      3: { x: 100, y: 280 },
      4: { x: 250, y: 280 },
      5: { x: 450, y: 280 },
      6: { x: 600, y: 280 },
      7: { x: 150, y: 410 },
      8: { x: 350, y: 410 },
      9: { x: 550, y: 410 }
    },
    edges: [
      { from: 0, to: 1 },
      { from: 0, to: 2 },
      { from: 1, to: 3 },
      { from: 1, to: 4 },
      { from: 2, to: 5 },
      { from: 2, to: 6 },
      { from: 3, to: 7 },
      { from: 4, to: 7 },
      { from: 4, to: 8 },
      { from: 5, to: 8 },
      { from: 6, to: 9 }
    ],
    adjacency: {
      0: [1, 2],
      1: [0, 3, 4],
      2: [0, 5, 6],
      3: [1, 7],
      4: [1, 7, 8],
      5: [2, 8],
      6: [2, 9],
      7: [3, 4],
      8: [4, 5],
      9: [6]
    }
  }
}

const currentNodes = ref({ ...graphConfigs.medium.nodes })
const currentEdges = ref([...graphConfigs.medium.edges])
const currentAdjacency = ref({ ...graphConfigs.medium.adjacency })

const steps = ref([])
const currentStep = ref(0)
const isPlaying = ref(false)
const speed = ref(1000)
let playInterval = null

const codeLines = [
  'void bfs(int start) {',
  '  queue<int> q;',
  '  vector<bool> visited(n, false); ',
  '  vector<int> dist(n, INF);',
  '  ',
  '  q.push(start); ',
  '  visited[start] = true;',
  '  dist[start] = 0;',
  '  ',
  '  while (!q.empty()) {',
  '    int u = q.front();',
  '    q.pop();',
  '    ',
  '    for (int v : adj[u]) {',
  '      if (!visited[v]) {',
  '        visited[v] = true;',
  '        dist[v] = dist[u] + 1;',
  '        q.push(v);',
  '      }',
  '    }',
  '  }',
  '}'
]

const currentCodeLine = ref(-1)

const currentStepData = computed(() => {
  if (steps.value.length === 0) return null
  return steps.value[currentStep.value]
})

const isEdgeExplored = (from, to) => {
  if (!currentStepData.value) return false
  const visited = currentStepData.value.visited
  return visited.has(from) && visited.has(to)
}

const isEdgeActive = (from, to) => {
  if (!currentStepData.value) return false
  const current = currentStepData.value.currentNode
  const exploring = currentStepData.value.exploringNeighbor
  return (current === from && exploring === to) || (current === to && exploring === from)
}

const generateSteps = () => {
  const stepsArray = []
  const visited = new Set()
  const queue = []
  const distance = {}

  const startNode = 0
  queue.push(startNode)
  visited.add(startNode)
  distance[startNode] = 0

  stepsArray.push({
    action: 'start',
    visited: new Set([startNode]),
    queue: [startNode],
    distance: { ...distance },
    currentNode: null,
    exploringNeighbor: null,
    explanation: `<strong>Inicialização:</strong><br>- Começamos no nó 0.<br>- Marcamos como visitado e adicionamos na fila.<br>- Distância do nó 0 para ele mesmo é 0.`,
    tip: 'BFS sempre visita nós em ordem crescente de distância da origem.',
    codeLine: 5
  })

  while (queue.length > 0) {
    const current = queue.shift()

    stepsArray.push({
      action: 'process',
      visited: new Set(visited),
      queue: [...queue],
      distance: { ...distance },
      currentNode: current,
      exploringNeighbor: null,
      explanation: `<strong>Processando nó ${current}:</strong><br>- Removemos o nó ${current} da frente da fila.<br>- Vamos explorar seus vizinhos: ${currentAdjacency.value[current].join(', ')}.`,
      tip: `Fila atual tem ${queue.length} nó(s). BFS usa FIFO (First In, First Out).`,
      codeLine: 10
    })

    for (const neighbor of currentAdjacency.value[current]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor)
        distance[neighbor] = distance[current] + 1
        queue.push(neighbor)

        stepsArray.push({
          action: 'enqueue',
          visited: new Set(visited),
          queue: [...queue],
          distance: { ...distance },
          currentNode: current,
          exploringNeighbor: neighbor,
          explanation: `<strong>Descobrindo vizinho ${neighbor}:</strong><br>- Nó ${neighbor} não foi visitado.<br>- Marcamos como visitado.<br>- Distância = ${distance[neighbor]} (distância do nó ${current} + 1).<br>- Adicionamos ${neighbor} na fila.`,
          tip: `A distância ${distance[neighbor]} é a menor possível, pois BFS explora por camadas.`,
          codeLine: 15
        })
      } else {
        stepsArray.push({
          action: 'skip',
          visited: new Set(visited),
          queue: [...queue],
          distance: { ...distance },
          currentNode: current,
          exploringNeighbor: neighbor,
          explanation: `<strong>Ignorando vizinho ${neighbor}:</strong><br>- Nó ${neighbor} já foi visitado.`,
          tip: 'BFS nunca processa o mesmo nó duas vezes.',
          codeLine: 14
        })
      }
    }
  }

  stepsArray.push({
    action: 'complete',
    visited: new Set(visited),
    queue: [],
    distance: { ...distance },
    currentNode: null,
    exploringNeighbor: null,
    explanation: `<strong>BFS Completo!</strong><br>- Fila vazia, todos os nós alcançáveis foram visitados.<br>- Total de nós visitados: ${visited.size}.`,
    tip: 'Complexidade: O(V + E) onde V = nós e E = arestas.',
    codeLine: 21
  })

  return stepsArray
}

const loadSmallGraph = () => {
  currentNodes.value = { ...graphConfigs.small.nodes }
  currentEdges.value = [...graphConfigs.small.edges]
  currentAdjacency.value = { ...graphConfigs.small.adjacency }
  reset()
}

const loadMediumGraph = () => {
  currentNodes.value = { ...graphConfigs.medium.nodes }
  currentEdges.value = [...graphConfigs.medium.edges]
  currentAdjacency.value = { ...graphConfigs.medium.adjacency }
  reset()
}

const loadLargeGraph = () => {
  currentNodes.value = { ...graphConfigs.large.nodes }
  currentEdges.value = [...graphConfigs.large.edges]
  currentAdjacency.value = { ...graphConfigs.large.adjacency }
  reset()
}

const reset = () => {
  currentStep.value = 0
  isPlaying.value = false
  if (playInterval) {
    clearInterval(playInterval)
    playInterval = null
  }
  steps.value = generateSteps()
}

const togglePlay = () => {
  if (isPlaying.value) {
    isPlaying.value = false
    if (playInterval) {
      clearInterval(playInterval)
      playInterval = null
    }
  } else {
    isPlaying.value = true
    playInterval = setInterval(() => {
      if (currentStep.value < steps.value.length - 1) {
        currentStep.value++
      } else {
        isPlaying.value = false
        clearInterval(playInterval)
        playInterval = null
      }
    }, speed.value)
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

watch(speed, () => {
  if (isPlaying.value) {
    clearInterval(playInterval)
    playInterval = setInterval(() => {
      if (currentStep.value < steps.value.length - 1) {
        currentStep.value++
      } else {
        isPlaying.value = false
        clearInterval(playInterval)
        playInterval = null
      }
    }, speed.value)
  }
})

watch(currentStepData, (newData) => {
  if (newData) {
    currentCodeLine.value = newData.codeLine
  }
})

// Initialize
steps.value = generateSteps()

</script>

<style scoped>
.bfs-enhanced {
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

.speed-control {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.speed-control input {
  width: 100%;
}

.speed-label {
  text-align: center;
  font-weight: var(--font-weight-bold);
  color: var(--color-primary-600);
  margin-top: var(--spacing-sm);
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
  grid-template-columns: 2fr 1fr;
  gap: var(--spacing-lg);
  align-items: start;
  margin-bottom: var(--spacing-lg);
}

.status-and-explanation {
    display: flex;
    flex-direction: column;
    gap: var(--spacing-lg);
}

.status-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: var(--spacing-md);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--spacing-md);
  border-radius: var(--radius-md);
  text-align: center;
  border: 1px solid var(--color-border);
}

.stat-label {
  font-size: var(--text-sm);
  color: var(--color-text-secondary);
  margin-bottom: var(--spacing-xs);
  display: block;
}

.stat-value {
  font-size: var(--text-lg);
  font-weight: var(--font-weight-bold);
  color: var(--color-text-primary);
  font-family: var(--font-mono);
}

.queue-display {
    font-size: var(--text-base);
    word-break: break-all;
}

.visualization {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
}

.graph-svg {
  display: block;
  width: 100%;
  height: auto;
  background: var(--color-background-alt);
  border-radius: var(--radius-md);
  border: 1px solid var(--color-border);
}

.edge {
  stroke: var(--color-border-strong);
  stroke-width: 2;
  transition: all 0.3s;
}

.edge.explored {
  stroke: var(--color-secondary-400);
  stroke-width: 3;
}

.edge.active {
  stroke: var(--color-accent-500);
  stroke-width: 4;
}

.node {
  fill: var(--color-surface);
  stroke: var(--color-neutral-700);
  stroke-width: 2;
  transition: all 0.3s ease;
  cursor: pointer;
}

.node.start {
  stroke: var(--color-primary-500);
  stroke-width: 3;
}

.node.inQueue {
  fill: var(--color-accent-100);
  stroke: var(--color-accent-500);
}

.node.current {
  fill: var(--color-accent-400);
  stroke: var(--color-accent-700);
  stroke-width: 4;
  animation: pulse 1s infinite;
}

.node.visited {
  fill: var(--color-secondary-500);
  stroke: var(--color-secondary-700);
}

@keyframes pulse {
  0%, 100% { r: 30; }
  50% { r: 33; }
}

.node-label {
  font-size: var(--text-lg);
  font-weight: var(--font-weight-bold);
  fill: var(--color-text-primary);
  pointer-events: none;
}

.node.visited + .node-label {
  fill: var(--color-text-inverse);
}

.distance-label {
  font-size: var(--text-sm);
  font-weight: var(--font-weight-semibold);
  fill: var(--color-primary-600);
  pointer-events: none;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-md);
  justify-content: center;
  margin-top: var(--spacing-lg);
}

.legend-item {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  font-size: var(--text-sm);
}

.legend-box {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 2px solid var(--color-neutral-700);
}

.legend-box.start { border-color: var(--color-primary-500); border-width: 3px; }
.legend-box.unvisited { background: var(--color-surface); }
.legend-box.in-queue { background: var(--color-accent-100); border-color: var(--color-accent-500); }
.legend-box.current-node { background: var(--color-accent-400); border-color: var(--color-accent-700); }
.legend-box.visited-node { background: var(--color-secondary-500); border-color: var(--color-secondary-700); }

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
  margin-top: var(--spacing-lg);
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

@media (max-width: 1200px) {
  .main-content-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 600px) {
  .controls-grid {
    grid-template-columns: 1fr;
  }
}
</style>