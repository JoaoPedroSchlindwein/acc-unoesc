<template>
  <div>
    <h1>Problemas Resolvidos</h1>
    <p class="intro">
      Soluções interativas de problemas da Maratona SBC aplicando o Framework ACC.
    </p>

    <div class="problems-grid">
      <div
        v-for="problem in problems"
        :key="problem.id"
        class="problem-card card card-interactive"
        @click="selectProblem(problem)"
      >
        <div class="problem-header">
          <h3>{{ problem.name }}</h3>
          <span class="badge badge-primary">{{ problem.category }}</span>
        </div>
        <div class="problem-info">
          <span class="year">{{ problem.year }}</span>
          <span class="badge difficulty" :class="problem.difficulty.toLowerCase()">{{ problem.difficulty }}</span>
        </div>
        <p class="summary">{{ problem.summary }}</p>
      </div>
    </div>

    <div v-if="selected" class="detail-section section">
      <h2>{{ selected.name }}</h2>
      <div class="problem-meta">
        <span class="meta-item">Ano: {{ selected.year }}</span>
        <span class="meta-item">Categoria: {{ selected.category }}</span>
        <span class="meta-item badge" :class="selected.difficulty.toLowerCase()">Dificuldade: {{ selected.difficulty }}</span>
      </div>

      <div class="acc-framework">
        <div class="acc-step" :class="{ active: accStep === 'algorithm' }" @click="accStep = 'algorithm'">
          <h3>1. Algoritmo</h3>
          <p>Conceituação e Intuição</p>
        </div>
        <div class="acc-step" :class="{ active: accStep === 'code' }" @click="accStep = 'code'">
          <h3>2. Código</h3>
          <p>Implementação</p>
        </div>
        <div class="acc-step" :class="{ active: accStep === 'conquest' }" @click="accStep = 'conquest'">
          <h3>3. Conquista</h3>
          <p>Aplicação e Teste</p>
        </div>
      </div>

      <div class="content-area">
        <div v-if="accStep === 'algorithm'" class="algorithm-explanation">
          <h3>Enunciado</h3>
          <div v-html="selected.statement"></div>

          <h3>Análise</h3>
          <div v-html="selected.analysis"></div>
        </div>

        <div v-if="accStep === 'code'" class="code-implementation">
          <h3>Implementação em C++</h3>
          <pre><code>{{ selected.code }}</code></pre>

          <div class="complexity-analysis">
            <h4>Análise de Complexidade</h4>
            <p><strong>Tempo:</strong> {{ selected.timeComplexity }}</p>
            <p><strong>Espaço:</strong> {{ selected.spaceComplexity }}</p>
          </div>
        </div>

        <div v-if="accStep === 'conquest'" class="conquest-section">
          <h3>Teste Interativo</h3>
          <component :is="selected.component" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, markRaw } from 'vue'
import AtencaoReuniaoSolver from '../components/problems/AtencaoReuniaoSolver.vue'
import GinasticaSolver from '../components/problems/GinasticaSolver.vue'
import ManiaParSolver from '../components/problems/ManiaParSolver.vue'

const accStep = ref('algorithm')
const selected = ref(null)

const problems = [
  {
    id: 1,
    name: 'Atenção à Reunião',
    year: '2024',
    category: 'Ad-Hoc',
    difficulty: 'Fácil',
    summary: 'Calcular o tempo máximo de discurso em uma reunião com intervalos',
    statement: `
      <p>Vinicius participa de uma reunião com N diretores, onde cada diretor discursará pelo mesmo tempo.
      Entre dois discursos consecutivos há um intervalo de 1 minuto.</p>
      <p>Dado que a reunião deve durar no máximo K minutos, determine o tempo máximo que cada diretor pode falar.</p>
      <p><strong>Entrada:</strong></p>
      <ul>
        <li>N: número de diretores (1 ≤ N ≤ 100)</li>
        <li>K: duração máxima da reunião em minutos (1 ≤ K ≤ 1000)</li>
      </ul>
    `,
    analysis: `
      <p>A modelagem matemática é fundamental:</p>
      <ul>
        <li>Tempo de discursos: N × T</li>
        <li>Tempo de intervalos: (N-1) × 1 = N - 1</li>
        <li>Tempo total: N × T + (N-1) ≤ K</li>
      </ul>
      <p>Resolvendo: T = ⌊(K - N + 1) / N⌋</p>
    `,
    code: `#include <bits/stdc++.h>
using namespace std;

int main() {
    int N, K;
    cin >> N >> K;

    // Tempo máximo = (K - intervalos) / N
    int tempoMaximo = (K - N + 1) / N;

    cout << tempoMaximo << endl;

    return 0;
}`,
    timeComplexity: 'O(1)',
    spaceComplexity: 'O(1)',
    component: markRaw(AtencaoReuniaoSolver)
  },
  {
    id: 2,
    name: 'Ginástica',
    year: '2017',
    category: 'Programação Dinâmica',
    difficulty: 'Médio',
    summary: 'Contar sequências válidas de níveis de dificuldade em exercícios',
    statement: `
      <p>Conte quantas sequências de T minutos são possíveis, onde:</p>
      <ul>
        <li>Cada minuto tem dificuldade entre M e N</li>
        <li>O nível deve mudar ±1 a cada minuto</li>
        <li>Não pode sair do intervalo [M, N]</li>
      </ul>
      <p>Retorne o resultado módulo 10⁹ + 7</p>
    `,
    analysis: `
      <p>Usamos Programação Dinâmica para evitar força bruta (2^49 casos):</p>
      <ul>
        <li>Estado: dp[t][d] = sequências que terminam no minuto t, nível d</li>
        <li>Transição: dp[t][d] vem de dp[t-1][d-1] e dp[t-1][d+1]</li>
        <li>Base: dp[1][d] = 1 para todo d em [M, N]</li>
      </ul>
      <p>A resposta é a soma de dp[T][d] para todo d.</p>
    `,
    code: `#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1000000007;

int main() {
    int T, M, N;
    cin >> T >> M >> N;

    int size = N - M + 1;
    vector<long long> dp(size, 1);

    for (int t = 2; t <= T; ++t) {
        vector<long long> dp_novo(size, 0);
        for (int i = 0; i < size; ++i) {
            int d = M + i;
            if (d == M) {
                dp_novo[i] = dp[i + 1];
            } else if (d == N) {
                dp_novo[i] = dp[i - 1];
            } else {
                dp_novo[i] = (dp[i - 1] + dp[i + 1]) % MOD;
            }
        }
        dp = dp_novo;
    }

    long long result = 0;
    for (int i = 0; i < size; ++i) {
        result = (result + dp[i]) % MOD;
    }

    cout << result << endl;

    return 0;
}`,
    timeComplexity: 'O(T × (N-M))',
    spaceComplexity: 'O(N-M)',
    component: markRaw(GinasticaSolver)
  },
  {
    id: 3,
    name: 'Mania de Par',
    year: '2015',
    category: 'Grafos',
    difficulty: 'Médio',
    summary: 'Encontrar o menor caminho com número par de arestas usando Dijkstra',
    statement: `
      <p>Patrícia tem mania de que tudo seja par. Ela só viaja por caminhos onde
      o número total de pedágios seja PAR.</p>
      <p>Dado um grafo com C cidades e V estradas bidirecionais, cada uma com um custo,
      encontre o menor custo para ir da cidade 1 até a cidade C, passando por um
      número PAR de pedágios.</p>
      <p><strong>Entrada:</strong></p>
      <ul>
        <li>C: número de cidades (2 ≤ C ≤ 10.000)</li>
        <li>V: número de estradas (0 ≤ V ≤ 50.000)</li>
        <li>V linhas com: u v g (cidades u e v conectadas com custo g)</li>
      </ul>
      <p><strong>Saída:</strong> Menor custo com número par de pedágios, ou -1 se impossível</p>
    `,
    analysis: `
      <p>O Dijkstra tradicional não rastreia o número de arestas. A solução é
      <strong>duplicar o espaço de estados</strong>:</p>
      <ul>
        <li>Estado (v, PAR): menor custo para v com número par de arestas</li>
        <li>Estado (v, ÍMPAR): menor custo para v com número ímpar de arestas</li>
      </ul>
      <p><strong>Transição:</strong> Uma aresta u→v inverte a paridade:</p>
      <ul>
        <li>(u, PAR) + aresta → (v, ÍMPAR)</li>
        <li>(u, ÍMPAR) + aresta → (v, PAR)</li>
      </ul>
      <p>Iniciamos em (1, PAR) com custo 0 e buscamos dist[C][PAR].</p>
    `,
    code: `#include <iostream>
#include <vector>
#include <queue>
#include <tuple>

using ll = long long;
using state = std::tuple<ll, int, int>; // {custo, vertice, paridade}

const int MAXC = 10005;
const ll INFINITO = 1e18;

int C, V;
std::vector<std::pair<int, int>> adj[MAXC];
ll dist[MAXC][2]; // dist[v][0] -> par, dist[v][1] -> impar

void dijkstra() {
    for (int i = 1; i <= C; ++i) {
        dist[i][0] = dist[i][1] = INFINITO;
    }

    std::priority_queue<state, std::vector<state>, std::greater<state>> pq;

    dist[1][0] = 0;
    pq.emplace(0, 1, 0);

    while (!pq.empty()) {
        auto [d_u, u, paridade_u] = pq.top();
        pq.pop();

        if (d_u > dist[u][paridade_u]) continue;

        for (auto& edge : adj[u]) {
            int v = edge.first;
            int w = edge.second;
            int nova_paridade = !paridade_u;

            if (dist[u][paridade_u] + w < dist[v][nova_paridade]) {
                dist[v][nova_paridade] = dist[u][paridade_u] + w;
                pq.emplace(dist[v][nova_paridade], v, nova_paridade);
            }
        }
    }
}

int main() {
    std::cin >> C >> V;
    for (int i = 0; i < V; ++i) {
        int u, v, g;
        std::cin >> u >> v >> g;
        adj[u].push_back({v, g});
        adj[v].push_back({u, g});
    }

    dijkstra();

    ll resultado = dist[C][0];
    std::cout << (resultado == INFINITO ? -1 : resultado) << std::endl;

    return 0;
}`,
    timeComplexity: 'O(V log C)',
    spaceComplexity: 'O(C + V)',
    component: markRaw(ManiaParSolver)
  }
]

const selectProblem = (problem) => {
  selected.value = problem
  accStep.value = 'algorithm'
}
</script>

<style scoped>
.intro {
  font-size: var(--text-lg);
  color: var(--color-text-secondary);
  margin-bottom: var(--spacing-2xl);
  max-width: 800px;
}

.problems-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: var(--spacing-lg);
  margin-bottom: var(--spacing-3xl);
}

.problem-card {
  padding: var(--spacing-lg);
}

.problem-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: var(--spacing-sm);
}

.problem-header h3 {
  color: var(--color-text-primary);
  margin: 0;
  font-size: var(--text-xl);
}

.problem-info {
  display: flex;
  gap: var(--spacing-md);
  align-items: center;
  margin-bottom: var(--spacing-md);
}

.year {
  color: var(--color-text-tertiary);
  font-size: var(--text-sm);
}

.badge.difficulty {
  font-size: var(--text-xs);
}

.badge.fácil {
  background-color: var(--color-secondary-100);
  color: var(--color-secondary-800);
}

.badge.médio {
  background-color: var(--color-accent-100);
  color: var(--color-accent-800);
}

.badge.difícil {
  background-color: var(--color-error-100);
  color: var(--color-error-700);
}

.summary {
  color: var(--color-text-secondary);
  font-size: var(--text-sm);
}

.detail-section {
  background: var(--color-surface);
  padding: var(--spacing-xl);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-base);
}

.detail-section h2 {
  margin-bottom: var(--spacing-sm);
}

.problem-meta {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-lg);
  margin-bottom: var(--spacing-xl);
  padding-bottom: var(--spacing-lg);
  border-bottom: 1px solid var(--color-border);
  font-size: var(--text-sm);
}

.meta-item {
  color: var(--color-text-secondary);
  display: inline-flex;
  align-items: center;
}

.acc-framework {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: var(--spacing-md);
  margin-bottom: var(--spacing-xl);
}

.acc-step {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  cursor: pointer;
  border: 1px solid var(--color-border);
  transition: all var(--transition-base);
}

.acc-step:hover {
  border-color: var(--color-primary-300);
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
}

.acc-step.active {
  background: linear-gradient(135deg, var(--color-primary-600) 0%, var(--color-primary-700) 100%);
  color: var(--color-text-inverse);
  border-color: var(--color-primary-700);
  box-shadow: var(--shadow-lg);
}

.acc-step.active h3,
.acc-step.active p {
  color: var(--color-text-inverse);
}

.acc-step h3 {
  margin-bottom: var(--spacing-sm);
  font-size: var(--text-lg);
}

.acc-step p {
  margin: 0;
  font-size: var(--text-sm);
  opacity: 0.9;
}

.content-area {
  background: var(--color-background-alt);
  padding: var(--spacing-xl);
  border-radius: var(--radius-md);
  min-height: 400px;
}

.algorithm-explanation h3,
.code-implementation h3,
.conquest-section h3 {
  color: var(--color-text-primary);
  margin-top: var(--spacing-lg);
  margin-bottom: var(--spacing-md);
}

.algorithm-explanation h3:first-child,
.code-implementation h3:first-child,
.conquest-section h3:first-child {
  margin-top: 0;
}

.code-implementation pre {
  background: var(--color-neutral-900);
  color: var(--color-neutral-100);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  overflow-x: auto;
  font-family: var(--font-mono);
  margin: var(--spacing-lg) 0;
}

.complexity-analysis {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  margin-top: var(--spacing-lg);
  border: 1px solid var(--color-border);
}

.complexity-analysis h4 {
  color: var(--color-text-primary);
  margin-bottom: var(--spacing-sm);
}

.complexity-analysis p {
  margin: 0.5rem 0;
  font-family: var(--font-mono);
  color: var(--color-secondary-700);
}
</style>