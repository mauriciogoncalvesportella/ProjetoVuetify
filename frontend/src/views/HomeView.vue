<script setup lang="ts">
// Importações
import DefaultLayout from '@/layouts/DefaultLayout.vue'
import { ref, computed, onMounted } from 'vue'
import { Line, Bar, Pie } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  ArcElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'
import * as XLSX from 'xlsx'
import Dashboard from './Dashboard.vue'
import router from '@/router'

// Registra componentes Chart.js
ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  ArcElement,
  Title,
  Tooltip,
  Legend
)

// Interface para notificações
interface Notificacao {
  id: number
  texto: string
  tipo: string
}

// Computed property para filtrar notificações
const visibleNotifications = computed(() => {
  if(dashboardConfig.value.mostrarNotificacoes) {
    return notificacoes.value
  }
  return []
})

// Estados reativos
const periodoSelecionado = ref('mes')
const tipoGrafico = ref('line')
const showCustomizeDialog = ref(false)
const showDetails = ref(false)
const notificacoes = ref<Notificacao[]>([])
const fileInput = ref<HTMLInputElement | null>(null)
const modoComparacao = ref(false)

// Estado para controle de visibilidade dos componentes
const dashboardConfig = ref({
  mostrarKPIs: true,         // Controla visibilidade dos KPIs
  mostrarComparativo: true,  // Controla visibilidade do botão comparar
  mostrarNotificacoes: true  // Controla visibilidade das notificações
})

// Função para salvar configurações
const salvarConfiguracoes = () => {
  // Salva configurações no localStorage para persistência
  localStorage.setItem('dashboardConfig', JSON.stringify(dashboardConfig.value))

  // Adiciona notificação
  notificacoes.value.push({
    id: Date.now(),
    texto: 'Configurações salvas com sucesso',
    tipo: 'sucesso'
  })

  // Fecha o diálogo
  showCustomizeDialog.value = false
}

// Função para restaurar as configurações padrão
const restaurarPadroes = () => {
  dashboardConfig.value = {
    mostrarKPIs: true,
    mostrarComparativo: true,
    mostrarNotificacoes: true
  }

  // Salva as configurações padrão
  localStorage.setItem('dashboardConfig', JSON.stringify(dashboardConfig.value))

  // Adiciona notificação
  notificacoes.value.push({
    id: Date.now(),
    texto: 'Configurações padrão restauradas',
    tipo: 'info'
  })

  // Fecha o diálogo
  showCustomizeDialog.value = false
}

// Dados originais para comparação
const dadosOriginais = ref({
  labels: [] as string[],
  datasets: [] as any[]
})

// KPIs
const kpis = ref({
  crescimento: '15%',
  previsao: 'R$ 65.2k',
  metaAtingida: '87%'
})

// Cards de estatísticas
const statCards = [
  {
    title: 'Usuários',
    value: '1,285',
    change: '+12%',
    icon: 'mdi-account-group',
    color: 'primary'
  },
  {
    title: 'Vendas',
    value: 'R$ 52.5k',
    change: '+8%',
    icon: 'mdi-chart-line',
    color: 'success'
  },
  {
    title: 'Projetos',
    value: '24',
    change: '5 concluídos',
    icon: 'mdi-folder',
    color: 'info'
  },
  {
    title: 'Horas',
    value: '160h',
    change: 'Este mês',
    icon: 'mdi-clock',
    color: 'warning'
  }
]

// Dados do gráfico
const chartData = ref({
  labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'],
  datasets: [{
    label: 'Vendas 2025',
    data: [30, 45, 57, 48, 65, 59],
    borderColor: '#1867C0',
    backgroundColor: 'rgba(24, 103, 192, 0.2)',
    tension: 0.1
  }]
})

// Opções do gráfico
const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: 'top' as const,
    },
    title: {
      display: true,
      text: 'Desempenho de Vendas'
    }
  }
}

// Função para abrir diálogo de detalhes
const verDetalhes = () => {
  showDetails.value = true
}

// Função para exportar Excel
const exportarParaExcel = () => {
  const ws = XLSX.utils.json_to_sheet(chartData.value.datasets[0].data.map((valor, index) => ({
    Mês: chartData.value.labels[index],
    Valor: valor
  })))

  const wb = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(wb, ws, 'Dados')
  XLSX.writeFile(wb, 'relatorio_vendas.xlsx')
}

// Função para filtrar por período
const filtrarPorPeriodo = (periodo: string) => {
  periodoSelecionado.value = periodo

  const dados = {
    mes: [30, 45, 57, 48, 65, 59],
    trimestre: [132, 170, 189],
    ano: [580, 720]
  }

  const labels = {
    mes: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'],
    trimestre: ['T1', 'T2', 'T3'],
    ano: ['2024', '2025']
  }

  chartData.value.datasets[0].data = dados[periodo as keyof typeof dados]
  chartData.value.labels = labels[periodo as keyof typeof labels]
}

// Função para mudar tipo de gráfico
const mudarTipoGrafico = (tipo: string) => {
  tipoGrafico.value = tipo
}

// Função para comparar períodos
const compararPeriodos = () => {
  modoComparacao.value = !modoComparacao.value

  if (modoComparacao.value) {
    dadosOriginais.value = JSON.parse(JSON.stringify(chartData.value))

    chartData.value = {
      labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun'],
      datasets: [
        {
          label: 'Atual (2025)',
          data: [30, 45, 57, 48, 65, 59],
          borderColor: '#1867C0',
          backgroundColor: 'rgba(24, 103, 192, 0.2)',
          tension: 0.1
        },
        {
          label: 'Anterior (2024)',
          data: [25, 38, 45, 40, 55, 48],
          borderColor: '#4CAF50',
          backgroundColor: 'rgba(76, 175, 80, 0.2)',
          tension: 0.1
        }
      ]
    }

    chartOptions.plugins.title.text = 'Comparação de Períodos'

    notificacoes.value.push({
      id: Date.now(),
      texto: 'Modo de comparação ativado',
      tipo: 'info'
    })
  } else {
    chartData.value = JSON.parse(JSON.stringify(dadosOriginais.value))
    chartOptions.plugins.title.text = 'Desempenho de Vendas'

    notificacoes.value.push({
      id: Date.now(),
      texto: 'Modo de comparação desativado',
      tipo: 'info'
    })
  }
}

// Função para trigger do input file
const triggerFileInput = () => {
  if (fileInput.value) {
    fileInput.value.click()
  }
}

// Função para processar arquivo Excel
const processExcel = async (event: Event) => {
  try {
    const input = event.target as HTMLInputElement
    const file = input?.files?.[0]
    if (!file) return

    const reader = new FileReader()

    reader.onload = (e) => {
      try {
        const data = new Uint8Array(e.target?.result as ArrayBuffer)
        const workbook = XLSX.read(data, {
          type: 'array',
          cellDates: true,
          dateNF: 'dd/mm/yyyy'
        })

        const firstSheet = workbook.Sheets[workbook.SheetNames[0]]
        const jsonData = XLSX.utils.sheet_to_json(firstSheet)

        console.log('Dados importados:', jsonData)

        const labels = jsonData.map((row: any) =>
          row.Mes || row.mes || row.MES || row.Data || row.data || row.DATA || '')
        const values = jsonData.map((row: any) => {
          const val = row.Valor || row.valor || row.VALOR ||
                     row.Vendas || row.vendas || row.VENDAS
          return typeof val === 'string' ? parseFloat(val) : val
        })

        if (labels.length === 0 || values.length === 0) {
          throw new Error('Dados inválidos no arquivo')
        }

        chartData.value = {
          labels,
          datasets: [{
            label: 'Dados Importados',
            data: values,
            borderColor: '#1867C0',
            backgroundColor: 'rgba(24, 103, 192, 0.2)',
            tension: 0.1
          }]
        }

        notificacoes.value.push({
          id: Date.now(),
          texto: 'Dados importados com sucesso!',
          tipo: 'sucesso'
        })

      } catch (error) {
        console.error('Erro ao processar arquivo:', error)
        notificacoes.value.push({
          id: Date.now(),
          texto: 'Erro ao processar o arquivo. Verifique o formato.',
          tipo: 'alerta'
        })
      }
    }

    reader.onerror = () => {
      notificacoes.value.push({
        id: Date.now(),
        texto: 'Erro ao ler o arquivo.',
        tipo: 'alerta'
      })
    }

    reader.readAsArrayBuffer(file)

  } catch (error) {
    console.error('Erro na importação:', error)
    notificacoes.value.push({
      id: Date.now(),
      texto: 'Erro na importação do arquivo.',
      tipo: 'alerta'
    })
  }
}

// Função para inicializar notificações
const checkNotificacoes = () => {
  notificacoes.value = [
    { id: 1, texto: 'Meta mensal atingida', tipo: 'sucesso' },
    { id: 2, texto: 'Novo relatório disponível', tipo: 'info' },
    { id: 3, texto: 'Alerta de desempenho', tipo: 'alerta' }
  ]
}

// logout login

const handleLogout = () => {
  localStorage.removeItem('authToken')
  localStorage.removeItem('user')
  router.push('/login')
}

// Inicialização no mount
onMounted(() => {
  checkNotificacoes()
  dadosOriginais.value = JSON.parse(JSON.stringify(chartData.value))

  // Carrega configurações salvas
  const savedConfig = localStorage.getItem('dashboardConfig')
  if (savedConfig) {
    dashboardConfig.value = JSON.parse(savedConfig)
  }
})
</script>

<template>
  <DefaultLayout>
    <!-- Cards de Estatísticas -->
    <v-row>
      <v-col v-for="card in statCards" :key="card.title" cols="12" sm="6" md="3">
        <v-card>
          <v-card-item>
            <template v-slot:prepend>
              <v-icon :color="card.color" size="large" :icon="card.icon"></v-icon>
            </template>
            <v-card-title>{{ card.title }}</v-card-title>
            <div class="text-h4 mb-1">{{ card.value }}</div>
            <div class="text-caption">{{ card.change }}</div>
          </v-card-item>
        </v-card>
      </v-col>
    </v-row>

    <!-- Barra de Ferramentas -->
    <v-row class="mb-4">
      <v-col>
        <v-card>
          <v-card-text class="d-flex align-center">
            <v-select
              v-model="periodoSelecionado"
              :items="[
                { title: 'Mensal', value: 'mes' },
                { title: 'Trimestral', value: 'trimestre' },
                { title: 'Anual', value: 'ano' }
              ]"
              label="Período"
              @update:model-value="filtrarPorPeriodo"
              class="mr-4"
              style="max-width: 200px"
              v-show="dashboardConfig.mostrarKPIs"
            ></v-select>

            <v-btn-group class="mr-4" v-show="dashboardConfig.mostrarKPIs">
              <v-btn :color="tipoGrafico === 'line' ? 'primary' : ''" @click="mudarTipoGrafico('line')">
                <v-icon>mdi-chart-line</v-icon>
              </v-btn>
              <v-btn :color="tipoGrafico === 'bar' ? 'primary' : ''" @click="mudarTipoGrafico('bar')">
                <v-icon>mdi-chart-bar</v-icon>
              </v-btn>
              <v-btn :color="tipoGrafico === 'pie' ? 'primary' : ''" @click="mudarTipoGrafico('pie')">
                <v-icon>mdi-chart-pie</v-icon>
              </v-btn>
            </v-btn-group>

            <v-btn color="success" class="mr-2" @click="exportarParaExcel" v-show="dashboardConfig.mostrarKPIs">
              <v-icon class="mr-2">mdi-file-excel</v-icon>
              Exportar
            </v-btn>

            <v-btn
              v-show="dashboardConfig.mostrarComparativo"
              color="info"
              class="mr-2"
              @click="compararPeriodos"
              :variant="modoComparacao ? 'flat' : 'elevated'"
            >
              <v-icon class="mr-2">mdi-compare</v-icon>
              {{ modoComparacao ? 'Desativar Comparação' : 'Comparar' }}
            </v-btn>

            <!-- Botão Personalizar sempre visível -->
            <v-btn color="primary" @click="showCustomizeDialog = true">
              <v-icon class="mr-2">mdi-cog</v-icon>
              Personalizar
            </v-btn>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- KPIs -->
    <v-row class="mb-4" v-show="dashboardConfig.mostrarKPIs">
      <v-col cols="12" md="4">
        <v-card>
          <v-card-text>
            <div class="text-subtitle-1">Taxa de Crescimento</div>
            <div class="text-h4 text-success">{{ kpis.crescimento }}</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card>
          <v-card-text>
            <div class="text-subtitle-1">Previsão Próximo Mês</div>
            <div class="text-h4">{{ kpis.previsao }}</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card>
          <v-card-text>
            <div class="text-subtitle-1">Meta Atingida</div>
            <div class="text-h4 text-primary">{{ kpis.metaAtingida }}</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Gráfico e Atividades -->
    <v-row class="mt-4">
      <v-col cols="12" md="8">
        <v-card>
          <v-card-title class="d-flex align-center">
            Desempenho Mensal
            <v-spacer></v-spacer>
            <v-btn variant="text" color="primary" @click="verDetalhes">
              Ver Detalhes
            </v-btn>
          </v-card-title>
          <v-card-text>
            <div style="height: 300px;">
              <component
                :is="tipoGrafico === 'pie' ? Pie : tipoGrafico === 'bar' ? Bar : Line"
                :data="chartData"
                :options="chartOptions"
              />
            </div>
          </v-card-text>
        </v-card>

        <!-- Diálogo de detalhes -->
        <v-dialog v-model="showDetails" max-width="800">
          <v-card>
            <v-card-title>Detalhes do Desempenho</v-card-title>
            <v-card-text>
              <Line :data="chartData" :options="chartOptions" />
              <v-divider class="my-4"></v-divider>
              <v-list>
                <v-list-item v-for="(dataset, index) in chartData.datasets" :key="index">
                  <v-list-item-title>
                    Média: {{ Math.round(dataset.data.reduce((a: any, b: any) => a + b) / dataset.data.length) }}
                  </v-list-item-title>
                  <v-list-item-subtitle>
                    Maior valor: {{ Math.max(...dataset.data) }}
                  </v-list-item-subtitle>
                </v-list-item>
              </v-list>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" @click="showDetails = false">Fechar</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-col>

      <!-- Seção de Atividades Recentes -->
      <v-col cols="12" md="4">
        <v-card>
          <v-card-title>Atividades Recentes</v-card-title>
          <v-list lines="two">
            <v-list-item>
              <v-list-item-title class="d-flex align-center">
                Novo projeto criado
                <v-btn class="ml-2" variant="text" color="primary" @click="triggerFileInput">
                  <v-icon class="mr-2">mdi-file-excel</v-icon>
                  Importar Excel
                </v-btn>
              </v-list-item-title>
              <input
                type="file"
                @change="processExcel"
                accept=".xlsx,.xls"
                style="display: none"
                ref="fileInput"
              >
              <v-list-item-subtitle>Há 5 minutos</v-list-item-subtitle>
            </v-list-item>
            <v-list-item
              prepend-icon="mdi-file-chart"
              title="Relatório mensal gerado"
              subtitle="Há 2 horas"
            ></v-list-item>
          </v-list>
        </v-card>
      </v-col>
    </v-row>

    <!-- Notificações -->
    <v-snackbar
      v-for="notif in visibleNotifications"
      :key="notif.id"
      :color="notif.tipo === 'sucesso' ? 'success' : notif.tipo === 'alerta' ? 'warning' : 'info'"
      :timeout="5000"
    >
      {{ notif.texto }}
    </v-snackbar>

    <!-- Diálogo de Personalização -->
    <v-dialog v-model="showCustomizeDialog" max-width="500">
      <v-card>
        <v-card-title class="text-h5 pb-2">
          Personalizar Dashboard
        </v-card-title>
        <v-divider></v-divider>
        <v-card-text class="pt-4">
          <v-list>
            <v-list-item>
              <v-switch
                v-model="dashboardConfig.mostrarKPIs"
                label="Mostrar KPIs"
                color="primary"
                hide-details
              ></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch
                v-model="dashboardConfig.mostrarComparativo"
                label="Mostrar Comparativo"
                color="primary"
                hide-details
              ></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch
                v-model="dashboardConfig.mostrarNotificacoes"
                label="Notificações"
                color="primary"
                hide-details
              ></v-switch>
            </v-list-item>
          </v-list>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-btn color="error" variant="outlined" @click="restaurarPadroes">
            <v-icon class="mr-2">mdi-refresh</v-icon>
            Restaurar Padrões
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="salvarConfiguracoes">
            <v-icon class="mr-2"<v-row>>mdi-content-save</v-icon>
            Salvar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </DefaultLayout>
</template>

<style scoped>
.v-card-title {
  font-size: 1.1rem;
}

.text-success {
  color: #4CAF50 !important;
}

.v-text-field {
  margin-top: 12px !important;
}

@media (min-width: 960px) {
  .v-container {
    padding: 24px;
  }

  .v-text-field {
    width: 300px;
  }
}

.v-btn {
  text-transform: none;
}

.v-card {
  transition: transform 0.2s;
}

.v-card:hover {
  transform: translateY(-2px);
}

.v-dialog {
  border-radius: 8px;
}

.v-snackbar {
  margin-bottom: 16px;
}

.chart-container {
  position: relative;
}
</style>
