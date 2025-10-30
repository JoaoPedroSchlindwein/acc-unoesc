<template>
  <div>
    <h1>Algoritmos Fundamentais</h1>
    <p class="intro">
      Algoritmos essenciais que aparecem recorrentemente em programação competitiva, agora com visualizações totalmente interativas.
    </p>

    <div class="algorithms-grid">
      <div
        v-for="algo in algorithms"
        :key="algo.name"
        class="algo-card card card-interactive"
        @click="selectAlgorithm(algo)"
      >
        <h3>{{ algo.icon }} {{ algo.name }}</h3>
        <p>{{ algo.description }}</p>
        <div class="complexity">{{ algo.complexity }}</div>
      </div>
    </div>

    <div v-if="selected" class="detail-section section">
      <h2>{{ selected.icon }} {{ selected.name }}</h2>

      <div class="tabs">
        <button
          :class="{ active: activeTab === 'visual' }"
          @click="activeTab = 'visual'"
        >
          <span class="tab-emoji">🎮</span> Visualização Interativa
        </button>
        <button
          :class="{ active: activeTab === 'explanation' }"
          @click="activeTab = 'explanation'"
        >
          <span class="tab-emoji">📚</span> Explicação
        </button>
        <button
          :class="{ active: activeTab === 'code' }"
          @click="activeTab = 'code'"
        >
          <span class="tab-emoji"> C++ </span> Código
        </button>
      </div>

      <div v-if="activeTab === 'visual'" class="visualization-area">
        <component :is="selected.component" />
      </div>

      <div v-if="activeTab === 'explanation'" class="explanation">
        <div v-html="selected.explanation"></div>
      </div>

      <div v-if="activeTab === 'code'" class="code-section">
        <pre><code>{{ selected.code }}</code></pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, markRaw } from 'vue'
import BinarySearchEnhanced from '../components/algorithms/BinarySearchEnhanced.vue'
import BFSEnhanced from '../components/algorithms/BFSEnhanced.vue'
import TwoPointersEnhanced from '../components/algorithms/TwoPointersEnhanced.vue'

const activeTab = ref('visual')
const selected = ref(null)

const algorithms = [
  {
    name: 'Busca Binária',
    icon: '🔍',
    description: 'Busca eficiente em arrays ordenados',
    complexity: 'Tempo: O(log n), Espaço: O(1)',
    component: markRaw(BinarySearchEnhanced),
    explanation: `
      <p>A busca binária elimina metade do espaço de busca a cada comparação.</p>
      <h3>Princípio:</h3>
      <ol>
        <li>Compare o elemento do meio com o alvo</li>
        <li>Se forem iguais, encontrou</li>
        <li>Se o alvo for menor, busque na metade esquerda</li>
        <li>Se o alvo for maior, busque na metade direita</li>
        <li>Repita até encontrar ou esgotar o espaço</li>
      </ol>
      <p>Para n = 1.000.000, são necessárias apenas 20 comparações!</p>
    `,
    code: `int buscaBinaria(const vector<int>& v, int alvo) {
    int esquerda = 0;
    int direita = v.size() - 1;

    while (esquerda <= direita) {
        int meio = esquerda + (direita - esquerda) / 2;

        if (v[meio] == alvo) {
            return meio;
        }

        if (v[meio] < alvo) {
            esquerda = meio + 1;
        } else {
            direita = meio - 1;
        }
    }

    return -1;
}`
  },
  {
    name: 'BFS (Busca em Largura)',
    icon: '🌊',
    description: 'Travessia de grafos camada por camada',
    complexity: 'Tempo: O(V + E), Espaço: O(V)',
    component: markRaw(BFSEnhanced),
    explanation: `
      <p>BFS explora o grafo camada por camada a partir de um vértice inicial.</p>
      <h3>Características:</h3>
      <ul>
        <li>Usa uma fila para gerenciar vértices</li>
        <li>Encontra o menor caminho em grafos não ponderados</li>
        <li>Visita vértices por nível de distância</li>
      </ul>
      <h3>Aplicações:</h3>
      <ul>
        <li>Menor caminho em grafos não ponderados</li>
        <li>Testar bipartição de grafos</li>
        <li>Encontrar componentes conexos</li>
      </ul>
    `,
    code: `void bfs(int start_node) {
    queue<int> q;
    q.push(start_node);
    visited[start_node] = true;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        for (int v : adj[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
}`
  },
  {
    name: 'Two Pointers',
    icon: '👉👉',
    description: 'Dois ponteiros para otimizar buscas',
    complexity: 'Tempo: O(n), Espaço: O(1)',
    component: markRaw(TwoPointersEnhanced),
    explanation: `
      <p>Two Pointers usa dois índices para percorrer estruturas, reduzindo O(n²) para O(n).</p>
      <h3>Padrões:</h3>
      <ul>
        <li><strong>Convergência:</strong> Ponteiros nas extremidades movendo-se ao centro</li>
        <li><strong>Perseguição:</strong> Ambos na mesma direção mantendo janela válida</li>
      </ul>
      <h3>Aplicações:</h3>
      <ul>
        <li>Encontrar par com soma específica</li>
        <li>Remover duplicatas in-place</li>
        <li>Mesclar arrays ordenados</li>
      </ul>
    `,
    code: `// Encontrar par com soma igual a alvo
pair<int,int> encontraPar(vector<int>& v, int alvo) {
    int esq = 0;
    int dir = v.size() - 1;

    while (esq < dir) {
        int soma = v[esq] + v[dir];

        if (soma == alvo) {
            return {esq, dir};
        } else if (soma < alvo) {
            esq++;
        } else {
            dir--;
        }
    }

    return {-1, -1};
}`
  }
]

const selectAlgorithm = (algo) => {
  selected.value = algo
  activeTab.value = 'visual'
}

</script>

<style scoped>
.intro {
  font-size: var(--text-lg);
  color: var(--color-text-secondary);
  margin-bottom: var(--spacing-2xl);
  max-width: 800px;
}

.algorithms-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: var(--spacing-lg);
  margin-bottom: var(--spacing-3xl);
}

.algo-card {
  padding: var(--spacing-lg);
  display: flex;
  flex-direction: column;
}

.algo-card h3 {
  color: var(--color-text-primary);
  margin-bottom: var(--spacing-sm);
  font-size: var(--text-xl);
}

.algo-card p {
  color: var(--color-text-secondary);
  margin-bottom: var(--spacing-md);
  flex-grow: 1;
}

.complexity {
  font-size: var(--text-sm);
  color: var(--color-secondary-700);
  font-family: var(--font-mono);
  font-weight: var(--font-weight-medium);
}

.detail-section {
  background: var(--color-surface);
  padding: var(--spacing-xl);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-base);
}

.detail-section h2 {
  margin-bottom: var(--spacing-lg);
}

.tabs {
  display: flex;
  gap: var(--spacing-md);
  margin-bottom: var(--spacing-xl);
  border-bottom: 1px solid var(--color-border);
  flex-wrap: wrap;
}

.tabs button {
  padding: var(--spacing-md) 0;
  border: none;
  background: transparent;
  color: var(--color-text-secondary);
  cursor: pointer;
  font-size: var(--text-base);
  font-weight: var(--font-weight-medium);
  transition: all var(--transition-base);
  border-bottom: 2px solid transparent;
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-sm);
}

.tabs button:hover:not(.active) {
  color: var(--color-text-primary);
}

.tabs button.active {
  color: var(--color-primary-600);
  font-weight: var(--font-weight-semibold);
  border-bottom-color: var(--color-primary-600);
}

.tab-emoji {
  font-size: 1.2em;
}

.visualization-area {
    border: 1px solid var(--color-border);
    border-radius: var(--radius-lg);
    overflow: hidden;
}

.explanation {
  line-height: var(--line-height-relaxed);
  color: var(--color-text-primary);
}

.explanation h3 {
  color: var(--color-text-primary);
  font-weight: var(--font-weight-semibold);
  margin-top: var(--spacing-lg);
  margin-bottom: var(--spacing-md);
}

.explanation ul,
.explanation ol {
  margin-left: var(--spacing-lg);
  margin-bottom: var(--spacing-md);
}

.explanation li {
  margin-bottom: var(--spacing-sm);
}

.code-section pre {
  background: var(--color-neutral-900);
  color: var(--color-neutral-100);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  overflow-x: auto;
  font-family: var(--font-mono);
}
</style>
