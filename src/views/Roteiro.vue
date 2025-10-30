<template>
  <div class="roteiro-container">
    <div class="page-header">
      <h1>Roteiro de Estudos</h1>
      <p class="intro">Sua trilha de aprendizado para a programação competitiva, do básico à resolução de problemas.</p>
    </div>

    <div class="timeline">
      <div v-for="(step, index) in roadmapSteps" :key="step.title" class="timeline-item">
        <div class="timeline-dot"></div>
        <div class="timeline-content card card-interactive">
          <router-link :to="step.link" class="timeline-link">
            <div class="step-header">
              <span class="step-number">Passo {{ index + 1 }}</span>
              <h3 class="step-title">{{ step.title }}</h3>
            </div>
            <p class="step-description">{{ step.description }}</p>
            <div class="step-tags">
              <span v-for="tag in step.tags" :key="tag" class="badge" :class="getTagClass(tag)">{{ tag }}</span>
            </div>
          </router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const roadmapSteps = [
  {
    title: 'Complexidade de Algoritmos',
    description: 'Entenda como medir a eficiência de um algoritmo. O Big O é o primeiro passo para escrever código que passa no tempo limite.',
    link: '/complexidade',
    tags: ['Teoria', 'Big O', 'Fundamental']
  },
  {
    title: 'Estruturas de Dados Básicas',
    description: 'Aprenda a usar as ferramentas essenciais da STL do C++: vector, queue e stack. Elas são os blocos de construção para algoritmos mais complexos.',
    link: '/estruturas',
    tags: ['Estruturas', 'STL', 'Essencial']
  },
  {
    title: 'Algoritmos Fundamentais',
    description: 'Domine técnicas poderosas como Busca Binária, Two Pointers e BFS. Elas aparecem em uma vasta gama de problemas.',
    link: '/algoritmos',
    tags: ['Algoritmos', 'Otimização']
  },
  {
    title: 'Resolvendo Problemas Ad-Hoc',
    description: 'Pratique a interpretação de enunciados e a lógica com problemas que não exigem algoritmos complexos, mas sim uma implementação cuidadosa.',
    link: '/problemas',
    tags: ['Prática', 'Ad-Hoc']
  },
  {
    title: 'Programação Dinâmica',
    description: 'Descubra como resolver problemas complexos quebrando-os em subproblemas menores e reutilizando soluções.',
    link: '/problemas', // Link to problems page, user can select DP problem
    tags: ['DP', 'Avançado']
  },
  {
    title: 'Grafos',
    description: 'Explore a modelagem de problemas com vértices e arestas, e aplique algoritmos como Dijkstra para encontrar o menor caminho.',
    link: '/problemas', // Link to problems page, user can select Graph problem
    tags: ['Grafos', 'Dijkstra', 'Avançado']
  }
];

const getTagClass = (tag) => {
  switch (tag.toLowerCase()) {
    case 'teoria':
    case 'fundamental':
      return 'badge-primary';
    case 'estruturas':
    case 'essencial':
      return 'badge-secondary';
    case 'algoritmos':
    case 'otimização':
      return 'badge-accent';
    case 'prática':
    case 'ad-hoc':
      return 'badge-primary'; // Reusing color
    default:
      return 'badge-neutral';
  }
};
</script>

<style scoped>
.roteiro-container {
  max-width: 900px;
  margin: 0 auto;
}

.page-header {
  text-align: center;
  margin-bottom: var(--spacing-3xl);
}

.intro {
  font-size: var(--text-lg);
  color: var(--color-text-secondary);
  max-width: 600px;
  margin: var(--spacing-md) auto 0;
}

.timeline {
  position: relative;
  padding: var(--spacing-lg) 0;
}

.timeline::before {
  content: '';
  position: absolute;
  top: 0;
  left: 12px;
  transform: translateX(-50%);
  width: 4px;
  height: 100%;
  background-color: var(--color-border);
  border-radius: 2px;
}

.timeline-item {
  position: relative;
  padding-left: calc(var(--spacing-xl) + 24px);
  margin-bottom: var(--spacing-xl);
}

.timeline-dot {
  content: '';
  position: absolute;
  top: 50%;
  left: 0;
  transform: translateY(-50%);
  width: 24px;
  height: 24px;
  background-color: var(--color-surface);
  border: 4px solid var(--color-primary-500);
  border-radius: 50%;
  z-index: 1;
}

.timeline-item:hover .timeline-dot {
    border-color: var(--color-secondary-500);
}

.timeline-content {
  padding: var(--spacing-lg);
}

.timeline-link {
  display: block;
  color: inherit;
  text-decoration: none;
}

.step-header {
  margin-bottom: var(--spacing-md);
}

.step-number {
  font-size: var(--text-sm);
  font-weight: var(--font-weight-bold);
  color: var(--color-primary-600);
  margin-bottom: var(--spacing-sm);
}

.step-title {
  font-size: var(--text-xl);
  margin: 0;
}

.step-description {
  font-size: var(--text-base);
  color: var(--color-text-secondary);
  line-height: var(--line-height-relaxed);
  margin-bottom: var(--spacing-lg);
}

.step-tags {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-sm);
}

.badge.badge-neutral {
    background-color: var(--color-neutral-200);
    color: var(--color-neutral-700);
}

@media (min-width: 768px) {
  .timeline::before {
    left: 50%;
  }
  .timeline-item {
    width: 50%;
    padding-left: var(--spacing-xl);
  }
  .timeline-item:nth-child(odd) {
    left: 0;
    padding-right: calc(var(--spacing-xl) + 24px);
    text-align: right;
  }
  .timeline-item:nth-child(even) {
    left: 50%;
    padding-left: calc(var(--spacing-xl) + 24px);
  }
  .timeline-item:nth-child(odd) .timeline-dot {
    right: -12px;
    left: auto;
  }
  .timeline-item:nth-child(even) .timeline-dot {
    left: -12px;
  }
}
</style>