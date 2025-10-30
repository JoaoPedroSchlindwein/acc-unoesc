<template>
  <div>
    <h1>Estruturas de Dados</h1>
    <p class="intro">
      Estruturas de dados da STL (Standard Template Library) do C++ são fundamentais
      para programação competitiva.
    </p>

    <div class="structures-grid">
      <div
        v-for="structure in structures"
        :key="structure.name"
        class="structure-card card card-interactive"
        @click="selectStructure(structure)"
      >
        <h3>{{ structure.icon }} {{ structure.name }}</h3>
        <p>{{ structure.description }}</p>
        <div class="complexity">{{ structure.complexity }}</div>
      </div>
    </div>

    <div v-if="selected" class="detail-section section">
      <h2>{{ selected.icon }} {{ selected.name }}</h2>

      <div class="tabs">
        <button
          :class="{ active: activeTab === 'visual' }"
          @click="activeTab = 'visual'"
        >
          Visualização
        </button>
        <button
          :class="{ active: activeTab === 'code' }"
          @click="activeTab = 'code'"
        >
          Código
        </button>
      </div>

      <div v-if="activeTab === 'visual'" class="visualization-area">
        <component :is="selected.component" />
      </div>

      <div v-if="activeTab === 'code'" class="code-section">
        <pre><code>{{ selected.code }}</code></pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, markRaw } from 'vue'
import VectorVisualization from '../components/structures/VectorVisualization.vue'
import QueueVisualization from '../components/structures/QueueVisualization.vue'
import StackVisualization from '../components/structures/StackVisualization.vue'

const activeTab = ref('visual')
const selected = ref(null)

const structures = [
  {
    name: 'Vector',
    icon: '📊',
    description: 'Array dinâmico com acesso O(1)',
    complexity: 'Acesso: O(1), Inserção: O(1) amortizado',
    component: markRaw(VectorVisualization),
    code: `vector<int> v;          // Declaração
v.push_back(10);        // Inserção: O(1) amortizado
int x = v[0];           // Acesso: O(1)
int tam = v.size();     // Tamanho: O(1)
v.pop_back();           // Remove último: O(1)`
  },
  {
    name: 'Queue',
    icon: '📥',
    description: 'Fila FIFO (First In, First Out)',
    complexity: 'Todas operações: O(1)',
    component: markRaw(QueueVisualization),
    code: `queue<int> q;           // Declaração
q.push(10);             // Inserção: O(1)
int x = q.front();      // Acesso ao primeiro: O(1)
q.pop();                // Remove primeiro: O(1)
bool vazia = q.empty(); // Verifica se vazia: O(1)`
  },
  {
    name: 'Stack',
    icon: '📚',
    description: 'Pilha LIFO (Last In, First Out)',
    complexity: 'Todas operações: O(1)',
    component: markRaw(StackVisualization),
    code: `stack<int> s;           // Declaração
s.push(10);             // Inserção: O(1)
int x = s.top();        // Acesso ao topo: O(1)
s.pop();                // Remove topo: O(1)
bool vazia = s.empty(); // Verifica se vazia: O(1)`
  },
  {
    name: 'Set',
    icon: '🔢',
    description: 'Conjunto ordenado sem duplicatas',
    complexity: 'Inserção/Busca/Remoção: O(log n)',
    component: markRaw(VectorVisualization), // Placeholder
    code: `set<int> s;             // Declaração
s.insert(10);           // Inserção: O(log n)
bool existe = s.count(5); // Busca: O(log n)
s.erase(10);            // Remoção: O(log n)`
  },
  {
    name: 'Map',
    icon: '🗺️',
    description: 'Associação chave-valor ordenada',
    complexity: 'Inserção/Busca/Remoção: O(log n)',
    component: markRaw(VectorVisualization), // Placeholder
    code: `map<int,int> m;         // Declaração
m[10] = 5;              // Inserção: O(log n)
int val = m[10];        // Acesso: O(log n)
bool existe = m.count(10); // Busca: O(log n)`
  },
  {
    name: 'Priority Queue',
    icon: '⬆️',
    description: 'Heap binário (maior/menor primeiro)',
    complexity: 'Inserção/Remoção: O(log n), Topo: O(1)',
    component: markRaw(VectorVisualization), // Placeholder
    code: `priority_queue<int> pq; // Max-heap
pq.push(10);            // Inserção: O(log n)
int maior = pq.top();   // Acesso ao maior: O(1)
pq.pop();               // Remove maior: O(log n)`
  }
]

const selectStructure = (structure) => {
  selected.value = structure
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

.structures-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: var(--spacing-lg);
  margin-bottom: var(--spacing-3xl);
}

.structure-card {
  padding: var(--spacing-lg);
  display: flex;
  flex-direction: column;
}

.structure-card h3 {
  color: var(--color-text-primary);
  margin-bottom: var(--spacing-sm);
  font-size: var(--text-xl);
}

.structure-card p {
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
}

.tabs button:hover:not(.active) {
  color: var(--color-text-primary);
}

.tabs button.active {
  color: var(--color-primary-600);
  font-weight: var(--font-weight-semibold);
  border-bottom-color: var(--color-primary-600);
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