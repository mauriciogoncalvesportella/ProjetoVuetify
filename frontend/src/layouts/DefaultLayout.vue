<script setup lang="ts">
import router from '@/router'
import { Title } from 'chart.js'
import { computed, onMounted, ref, watch } from 'vue'
import { useTheme } from 'vuetify'

// Definições de tipos
type BorderRadiusType = 'default' | 'rounded' | 'square'
type DensityType = 'compact' | 'default' | 'comfortable'
type SidebarPositionType = 'left' | 'right'
type ChartType = 'line' | 'bar' | 'pie'
type DashboardLayoutType = 'default' | 'compact' | 'expanded'

interface SystemSettings {
  // Interface
  sidebarPosition: SidebarPositionType
  menuCompact: boolean
  borderRadius: BorderRadiusType

  // Aparência
  density: DensityType
  fontScale: number
  animationsEnabled: boolean

  // Preferências
  dateFormat: string
  autoRefresh: boolean
  refreshInterval: number
  defaultView: string

  // Notificações
  emailNotifications: boolean
  pushNotifications: boolean
  notificationSound: boolean

  // Dashboard
  defaultChartType: ChartType
  showGridLines: boolean
  showLegend: boolean
  dashboardLayout: DashboardLayoutType
}

interface SystemNotification {
  id: number
  title: string
  message: string
  type: 'info' | 'success' | 'warning' | 'error'
  date: Date
  read: boolean
}

// Constantes
const borderRadiusMap: Record<BorderRadiusType, string> = {
  default: '4px',
  rounded: '12px',
  square: '0px'
} as const

// Estados reativos
const systemSettings = ref<SystemSettings>({
  sidebarPosition: 'left',
  menuCompact: false,
  borderRadius: 'default',
  density: 'default',
  fontScale: 1,
  animationsEnabled: true,
  dateFormat: 'dd/MM/yyyy',
  autoRefresh: true,
  refreshInterval: 5,
  defaultView: 'dashboard',
  emailNotifications: true,
  pushNotifications: true,
  notificationSound: true,
  defaultChartType: 'line',
  showGridLines: true,
  showLegend: true,
  dashboardLayout: 'default'
})

const notifications = ref<SystemNotification[]>([
  {
    id: 1,
    title: 'Nova Meta Atingida',
    message: 'Parabéns! Sua equipe atingiu a meta mensal de vendas.',
    type: 'success',
    date: new Date(2024, 1, 13, 14, 30),
    read: false
  },
  {
    id: 2,
    title: 'Relatório Pendente',
    message: 'O relatório mensal precisa ser finalizado até amanhã.',
    type: 'warning',
    date: new Date(2024, 1, 13, 10, 15),
    read: false
  },
  {
    id: 3,
    title: 'Novo Projeto Criado',
    message: 'O projeto "Data Analytics" foi criado com sucesso.',
    type: 'info',
    date: new Date(2024, 1, 12, 16, 45),
    read: true
  }
])

const showSettingsDialog = ref(false)
const showNotificationsDialog = ref(false)
const drawer = ref(false)
const search = ref('')
const theme = ref('light')
const vuetifyTheme = useTheme()

// Funções de configurações
const applyVisualSettings = () => {
  const app = document.querySelector('.v-application') as HTMLElement
  if (app) {
    app.setAttribute('data-density', systemSettings.value.density)
  }

  document.documentElement.style.fontSize = `${systemSettings.value.fontScale * 100}%`

  const borderRadius = systemSettings.value.borderRadius
  document.documentElement.style.setProperty('--v-border-radius', borderRadiusMap[borderRadius])

  if (systemSettings.value.sidebarPosition === 'right') {
    document.querySelector('.v-navigation-drawer')?.classList.add('v-navigation-drawer--right')
  } else {
    document.querySelector('.v-navigation-drawer')?.classList.remove('v-navigation-drawer--right')
  }
}

const applyNotificationSettings = () => {
  if (systemSettings.value.notificationSound) {
    console.log('Som de notificação ativado')
  }

  if (systemSettings.value.pushNotifications) {
    requestNotificationPermission()
  }
}

const requestNotificationPermission = async () => {
  if ('Notification' in window) {
    try {
      const permission = await Notification.requestPermission()
      if (permission === 'granted') {
        console.log('Notificações push ativadas')
      }
    } catch (error) {
      console.error('Erro ao solicitar permissão de notificação:', error)
    }
  }
}

const applyDashboardSettings = () => {
  const dashboardEvent = new CustomEvent('dashboard-settings-changed', {
    detail: {
      chartType: systemSettings.value.defaultChartType,
      layout: systemSettings.value.dashboardLayout,
      showGridLines: systemSettings.value.showGridLines,
      showLegend: systemSettings.value.showLegend
    }
  })
  window.dispatchEvent(dashboardEvent)
}

const saveSettings = () => {
  localStorage.setItem('systemSettings', JSON.stringify(systemSettings.value))
  applyVisualSettings()
  applyNotificationSettings()
  applyDashboardSettings()

  notifications.value.push({
    id: Date.now(),
    title: 'Configurações Salvas',
    message: 'Suas preferências foram atualizadas com sucesso.',
    type: 'success',
    date: new Date(),
    read: false
  })

  showSettingsDialog.value = false
}

const resetSettings = () => {
  systemSettings.value = {
    sidebarPosition: 'left',
    menuCompact: false,
    borderRadius: 'default',
    density: 'default',
    fontScale: 1,
    animationsEnabled: true,
    dateFormat: 'dd/MM/yyyy',
    autoRefresh: true,
    refreshInterval: 5,
    defaultView: 'dashboard',
    emailNotifications: true,
    pushNotifications: true,
    notificationSound: true,
    defaultChartType: 'line',
    showGridLines: true,
    showLegend: true,
    dashboardLayout: 'default'
  }
  saveSettings()
}

// Funções de notificação
const markAsRead = (notificationId: number) => {
  const notification = notifications.value.find(n => n.id === notificationId)
  if (notification) {
    notification.read = true
  }
}

const unreadCount = computed(() => notifications.value.filter(n => !n.read).length)

// Dados de navegação
const navigationItems = [
  { title: 'Dashboard', icon: 'mdi-view-dashboard', path: '/' },
  { title: 'Projetos', icon: 'mdi-folder', path: '/projects' },
  { title: 'Equipe', icon: 'mdi-account-group', path: '/team' },
  { title: 'Calendário', icon: 'mdi-calendar', path: '/calendar' },
  { title: 'Relatórios', icon: 'mdi-chart-box', path: '/reports' }
]

const userMenuItems = [
  {
    title: 'Perfil',
    icon: 'mdi-account-circle',
    action: 'profile',
    description: 'Visualize e edite seu perfil'
  },
  {
    title: 'Configurações',
    icon: 'mdi-cog',
    action: 'settings',
    description: 'Ajuste suas preferências'
  },
  {
    title: 'Notificações',
    icon: 'mdi-bell',
    action: 'notifications',
    description: 'Gerencie suas notificações',
    badge: unreadCount
  },
  { divider: true },
  {
    title: 'Sair',
    icon: 'mdi-logout',
    action: 'logout',
    description: 'Encerrar sessão'
  }
]

// Funções gerais
const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
  vuetifyTheme.global.name.value = theme.value
}

const handleLogout = async () => {
  localStorage.removeItem('authToken')
  localStorage.removeItem('user')
  await router.push('/login')
  window.location.reload()
}

const handleUserAction = async (action: string) => {
  switch (action) {
    case 'profile':
      console.log('Abrindo perfil do usuário')
      break
    case 'settings':
      showSettingsDialog.value = true
      break
    case 'notifications':
      showNotificationsDialog.value = true
      break
    case 'logout':
      await handleLogout()
      break
  }
}

// Watch e Lifecycle hooks
watch(theme, (newTheme) => {
  vuetifyTheme.global.name.value = newTheme
})

watch(systemSettings, () => {
  applyVisualSettings()
  applyNotificationSettings()
  applyDashboardSettings()
}, { deep: true })

onMounted(() => {
  const savedSettings = localStorage.getItem('systemSettings')
  if (savedSettings) {
    systemSettings.value = JSON.parse(savedSettings)
  }
  applyVisualSettings()
  applyNotificationSettings()
  applyDashboardSettings()
})
</script>

<template>
  <v-app :theme="theme">
    <!-- === BARRA SUPERIOR === -->
    <v-app-bar color="primary">
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>Data Company Sistemas</v-toolbar-title>
      <v-spacer></v-spacer>

      <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Buscar"
        hide-details
        density="compact"
        variant="outlined"
        class="hidden-sm-and-down mx-4"
        style="max-width: 300px"
      ></v-text-field>

      <v-btn icon @click="toggleTheme">
        <v-icon>
          {{ theme === 'light' ? 'mdi-weather-night' : 'mdi-weather-sunny' }}
        </v-icon>
      </v-btn>

      <v-btn icon class="mx-5" @click="showNotificationsDialog = true">
        <v-badge
          :content="unreadCount"
          :color="unreadCount > 0 ? 'error' : 'success'"
          :model-value="unreadCount > 0"
        >
          <v-icon>mdi-bell</v-icon>
        </v-badge>
      </v-btn>

      <v-menu location="bottom end">
        <template v-slot:activator="{ props }">
          <v-btn class="ml-2" v-bind="props">
            <v-avatar size="32" class="mr-2">
              <v-icon>mdi-account</v-icon>
            </v-avatar>
            <span class="hidden-sm-and-down">Usuário</span>
            <v-icon right>mdi-chevron-down</v-icon>
          </v-btn>
        </template>

        <v-list width="300">
          <template v-for="(item, index) in userMenuItems" :key="index">
            <v-divider v-if="item.divider" />
            <v-list-item
              v-else
              :value="index"
              @click="item.action && handleUserAction(item.action)"
            >
              <template v-slot:prepend>
                <v-icon :icon="item.icon"></v-icon>
              </template>
              <v-list-item-title>{{ item.title }}</v-list-item-title>
              <v-list-item-subtitle>{{ item.description }}</v-list-item-subtitle>
              <template v-if="item.badge" v-slot:append>
                <v-badge :content="item.badge" color="error"></v-badge>
              </template>
            </v-list-item>
          </template>
        </v-list>
      </v-menu>
    </v-app-bar>

    <!-- === MENU LATERAL === -->
    <v-navigation-drawer v-model="drawer" temporary>
      <v-list>
        <v-list-item
          v-for="item in navigationItems"
          :key="item.title"
          :to="item.path"
          :prepend-icon="item.icon"
          :title="item.title"
        ></v-list-item>
      </v-list>
    </v-navigation-drawer>

    <!-- === CONTEÚDO PRINCIPAL === -->
    <v-main>
      <v-container fluid>
        <slot></slot>
      </v-container>
    </v-main>

    <!-- === DIÁLOGO DE NOTIFICAÇÕES === -->
    <v-dialog v-model="showNotificationsDialog" max-width="500">
      <v-card>
        <v-card-title class="d-flex align-center">
          Notificações
          <v-spacer></v-spacer>
          <v-chip :color="unreadCount > 0 ? 'error' : 'success'" size="small">
            {{ unreadCount }} não lidas
          </v-chip>
        </v-card-title>

        <v-divider></v-divider>

        <v-card-text class="pa-0">
          <v-list>
            <v-list-item
              v-for="notification in notifications"
              :key="notification.id"
              :class="{ 'bg-grey-lighten-4': !notification.read }"
              @click="markAsRead(notification.id)"
            >
              <template v-slot:prepend>
                <v-icon :color="notification.type === 'success' ? 'success' :
                               notification.type === 'warning' ? 'warning' :
                               notification.type === 'error' ? 'error' : 'info'">
                  {{ notification.type === 'success' ? 'mdi-check-circle' :
                     notification.type === 'warning' ? 'mdi-alert' :
                     notification.type === 'error' ? 'mdi-alert-circle' : 'mdi-information' }}
                </v-icon>
              </template>

              <v-list-item-title :class="{ 'font-weight-bold': !notification.read }">
                {{ notification.title }}
              </v-list-item-title>

              <v-list-item-subtitle>
                {{ notification.message }}
              </v-list-item-subtitle>

              <v-list-item-subtitle class="text-caption text-grey">
                {{ new Date(notification.date).toLocaleString() }}
              </v-list-item-subtitle>

              <template v-slot:append>
                <v-badge
                  v-if="!notification.read"
                  dot
                  color="error"
                ></v-badge>
              </template>
            </v-list-item>
          </v-list>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="showNotificationsDialog = false">
            Fechar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- === DIÁLOGO DE CONFIGURAÇÕES === -->
    <v-dialog v-model="showSettingsDialog" max-width="700">
      <v-card>
        <v-card-title class="d-flex align-center">
          Configurações do Sistema
          <v-spacer></v-spacer>
          <v-btn icon @click="showSettingsDialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>

        <v-divider></v-divider>

        <v-card-text>
          <v-container>
            <v-row>
              <!-- Interface -->
              <v-col cols="12">
                <h3 class="text-h6 mb-3">Interface</h3>
                <v-row>
                  <v-col cols="12" md="6">
                    <v-select
                      v-model="systemSettings.sidebarPosition"
                      label="Posição do Menu Lateral"
                      :items="[
                        { title: 'Esquerda', value: 'left' },
                        { title: 'Direita', value: 'right' }
                      ]"
                      item-title="title"
                      item-value="value"
                    ></v-select>
                  </v-col>
                  <v-col cols="12" md="6">
                    <v-select
                      v-model="systemSettings.borderRadius"
                      label="Bordas dos Elementos"
                      :items="[
                        { title: 'Padrão', value: 'default' },
                        { title: 'Arredondadas', value: 'rounded' },
                        { title: 'Quadradas', value: 'square' }
                      ]"
                      item-title="title"
                      item-value="value"
                    ></v-select>
                  </v-col>
                </v-row>
              </v-col>

              <!-- Aparência -->
              <v-col cols="12">
                <h3 class="text-h6 mb-3">Aparência</h3>
                <v-row>
                  <v-col cols="12" md="6">
                    <v-select
                      v-model="systemSettings.density"
                      label="Densidade da Interface"
                      :items="[
                        { title: 'Compacta', value: 'compact' },
                        { title: 'Padrão', value: 'default' },
                        { title: 'Confortável', value: 'comfortable' }
                      ]"
                      item-title="title"
                      item-value="value"
                    ></v-select>
                  </v-col>
                  <v-col cols="12" md="6">
                    <v-slider
                      v-model="systemSettings.fontScale"
                      label="Escala da Fonte"
                      :min="0.9"
                      :max="1.1"
                      :step="0.1"
                      thumb-label
                    ></v-slider>
                  </v-col>
                </v-row>
              </v-col>

              <!-- Notificações -->
              <v-col cols="12">
                <h3 class="text-h6 mb-3">Notificações</h3>
                <v-row>
                  <v-col cols="12" md="4">
                    <v-switch
                      v-model="systemSettings.emailNotifications"
                      label="Notificações por E-mail"
                    ></v-switch>
                  </v-col>
                  <v-col cols="12" md="4">
                    <v-switch
                      v-model="systemSettings.pushNotifications"
                      label="Notificações Push"
                    ></v-switch>
                  </v-col>
                  <v-col cols="12" md="4">
                    <v-switch
                      v-model="systemSettings.notificationSound"
                      label="Som de Notificação"
                    ></v-switch>
                  </v-col>
                </v-row>
              </v-col>

              <!-- Dashboard -->
              <v-col cols="12">
                <h3 class="text-h6 mb-3">Dashboard</h3>
                <v-row>
                  <v-col cols="12" md="6">
                    <v-select
                      v-model="systemSettings.defaultChartType"
                      label="Tipo de Gráfico Padrão"
                      :items="[
                        { title: 'Linha', value: 'line' },
                        { title: 'Barra', value: 'bar' },
                        { title: 'Pizza', value: 'pie' }
                      ]"
                      item-title="title"
                      item-value="value"
                    ></v-select>
                  </v-col>
                  <v-col cols="12" md="6">
                    <v-select
                      v-model="systemSettings.dashboardLayout"
                      label="Layout do Dashboard"
                      :items="[
                        { title: 'Padrão', value: 'default' },
                        { title: 'Compacto', value: 'compact' },
                        { title: 'Expandido', value: 'expanded' }
                      ]"
                      item-title="title"
                      item-value="value"
                    ></v-select>
                  </v-col>
                </v-row>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-btn color="error" variant="text" @click="resetSettings">
            Restaurar Padrões
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="grey" variant="text" @click="showSettingsDialog = false">
            Cancelar
          </v-btn>
          <v-btn color="primary" @click="saveSettings">
            Salvar Alterações
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- === RODAPÉ === -->
    <v-footer app class="d-flex flex-column">
      <div class="w-100 d-flex align-center justify-space-between px-4">
        <span>&copy; {{ new Date().getFullYear() }} Data Company Sistemas</span>
      </div>
    </v-footer>
  </v-app>
</template>

<style>
/* Estilos base */
.v-app-bar {
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

.v-navigation-drawer {
  background-color: #f5f5f5;
}

.v-list-item {
  cursor: pointer;
  transition: background-color 0.2s;
}

.v-list-item:hover {
  background-color: rgba(0, 0, 0, 0.04);
}

/* Estilos para diferentes densidades */
.v-application[data-density="comfortable"] .v-btn {
  padding: 12px 24px;
}

.v-application[data-density="compact"] .v-btn {
  padding: 4px 12px;
}

/* Transições suaves para mudanças */
.v-application * {
  transition: all 0.3s ease;
}

/* Estilos para diferentes layouts do dashboard */
.dashboard-layout-compact .v-card {
  margin: 4px;
}

.dashboard-layout-expanded .v-card {
  margin: 16px;
}

/* Variáveis CSS personalizadas */
:root {
  --v-border-radius: 4px;
}

/* Aplicar border-radius personalizado */
.v-card,
.v-btn,
.v-text-field,
.v-select {
  border-radius: var(--v-border-radius) !important;
}
</style>
