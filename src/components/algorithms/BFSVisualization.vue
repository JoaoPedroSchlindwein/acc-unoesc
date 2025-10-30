<template>
  <div class="bfs-viz">
    <div class="controls">
      <button @click="startBFS" :disabled="isRunning" class="btn btn-primary">
        Iniciar BFS
      </button>
      <button @click="reset" class="btn btn-ghost">Reset</button>
      <label>
        Velocidade:
        <input v-model.number="speed" type="range" min="100" max="2000" step="100" />
        {{ 2100 - speed }}ms
      </label>
    </div>

    <div class="info">
      <span v-if="!isRunning && visited.size > 0" class="info-item">
        Visitados: {{ visited.size }} nós
      </span>
      <span v-if="isRunning" class="info-item">
        Processando nó: {{ currentNode }}
      </span>
      <span v-if="isRunning" class="info-item">
        Fila: [{{ queue.join(', ') }}]
      </span>
    </div>

    <div class="visualization">
      <svg width="600" height="400" class="graph-svg">
        <!-- Edges -->
        <g>
          <line
            v-for="edge in edges"
            :key="`${edge.from}-${edge.to}`"
            :x1="nodes[edge.from].x"
            :y1="nodes[edge.from].y"
            :x2="nodes[edge.to].x"
            :y2="nodes[edge.to].y"
            class="edge"
          />
        </g>

        <!-- Nodes -->
        <g>
          <g v-for="(node, id) in nodes" :key="id">
            <circle
              :cx="node.x"
              :cy="node.y"
              :r="25"
              class="node"
              :class="{
                visited: visited.has(id),
                current: currentNode === id,
                inQueue: queue.includes(id)
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
              v-if="distance[id] !== Infinity && distance[id] !== undefined"
              :x="node.x"
              :y="node.y - 40"
              text-anchor="middle"
              class="distance-label"
            >
              d={{ distance[id] }}
            </text>
          </g>
        </g>
      </svg>

      <div class="legend">
        <div class="legend-item">
          <div class="legend-box unvisited"></div>
          <span>Não visitado</span>
        </div>
        <div class="legend-item">
          <div class="legend-box in-queue"></div>
          <span>Na fila</span>
        </div>
        <div class="legend-item">
          <div class="legend-box current-node"></div>
          <span>Processando</span>
        </div>
        <div class="legend-item">
          <div class="legend-box visited-node"></div>
          <span>Visitado</span>
        </div>
      </div>
    </div>

    <div class="explanation">
      <h4>Como funciona</h4>
      <ol>
        <li>Inicia no nó 0 e o marca como visitado</li>
        <li>Adiciona o nó inicial na fila</li>
        <li>Enquanto a fila não estiver vazia:
          <ul>
            <li>Remove um nó da frente da fila</li>
            <li>Processa todos os vizinhos não visitados</li>
            <li>Marca vizinhos como visitados e adiciona na fila</li>
          </ul>
        </li>
        <li>A distância de cada nó é calculada automaticamente</li>
      </ol>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const isRunning = ref(false)
const currentNode = ref(null)
const queue = ref([])
const visited = ref(new Set())
const distance = ref({})
const speed = ref(1000)

const nodes = reactive({
  0: { x: 300, y: 50 },
  1: { x: 150, y: 150 },
  2: { x: 450, y: 150 },
  3: { x: 100, y: 300 },
  4: { x: 250, y: 300 },
  5: { x: 400, y: 300 },
  6: { x: 550, y: 300 }
})

const edges = [
  { from: 0, to: 1 },
  { from: 0, to: 2 },
  { from: 1, to: 3 },
  { from: 1, to: 4 },
  { from: 2, to: 5 },
  { from: 2, to: 6 },
  { from: 3, to: 4 }
]

const adjacencyList = reactive({
  0: [1, 2],
  1: [0, 3, 4],
  2: [0, 5, 6],
  3: [1, 4],
  4: [1, 3],
  5: [2],
  6: [2]
})

const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const startBFS = async () => {
  reset()
  isRunning.value = true

  const startNode = 0
  queue.value = [startNode]
  visited.value.add(startNode)
  distance.value[startNode] = 0

  while (queue.value.length > 0) {
    await delay(2100 - speed.value)

    currentNode.value = queue.value.shift()
    await delay(2100 - speed.value)

    for (const neighbor of adjacencyList[currentNode.value]) {
      if (!visited.value.has(neighbor)) {
        visited.value.add(neighbor)
        distance.value[neighbor] = distance.value[currentNode.value] + 1
        queue.value.push(neighbor)
        await delay(2100 - speed.value)
      }
    }

    currentNode.value = null
  }

  isRunning.value = false
}

const reset = () => {
  isRunning.value = false
  currentNode.value = null
  queue.value = []
  visited.value = new Set()
  distance.value = {}
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
}

.controls input[type="range"] {
  width: 150px;
}

.info {
  margin-bottom: var(--spacing-lg);
  min-height: 30px;
}

.info-item {
  background: var(--color-neutral-100);
  color: var(--color-text-secondary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-md);
  margin-right: var(--spacing-md);
  font-family: var(--font-mono);
  display: inline-block;
  margin-bottom: var(--spacing-sm);
}

.visualization {
  background: var(--color-surface);
  padding: var(--spacing-xl);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  margin-bottom: var(--spacing-lg);
}

.graph-svg {
  display: block;
  margin: 0 auto;
  background: var(--color-background-alt);
  border-radius: var(--radius-md);
}

.edge {
  stroke: var(--color-border-strong);
  stroke-width: 2;
}

.node {
  fill: var(--color-surface);
  stroke: var(--color-neutral-700);
  stroke-width: 2;
  transition: all 0.3s ease;
}

.node.inQueue {
  fill: var(--color-accent-100);
  stroke: var(--color-accent-500);
  stroke-width: 3;
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
  0%, 100% {
    r: 25;
  }
  50% {
    r: 28;
  }
}

.node-label {
  font-size: var(--text-base);
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
}

.legend {
  display: flex;
  gap: var(--spacing-lg);
  justify-content: center;
  margin-top: var(--spacing-xl);
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  font-size: var(--text-sm);
}

.legend-box {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  border: 2px solid var(--color-neutral-700);
}

.legend-box.unvisited {
  background: var(--color-surface);
}

.legend-box.in-queue {
  background: var(--color-accent-100);
  border-color: var(--color-accent-500);
}

.legend-box.current-node {
  background: var(--color-accent-400);
  border-color: var(--color-accent-700);
}

.legend-box.visited-node {
  background: var(--color-secondary-500);
  border-color: var(--color-secondary-700);
}

.explanation {
  background: var(--color-primary-50);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  border-left: 4px solid var(--color-primary-500);
}

.explanation h4 {
  color: var(--color-text-primary);
  margin-bottom: var(--spacing-md);
}

.explanation ol {
  margin-left: var(--spacing-lg);
  line-height: 1.8;
}

.explanation ul {
  margin-left: var(--spacing-lg);
  margin-top: var(--spacing-sm);
}
</style>