<script setup lang="ts">
import { onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

onMounted(() => {
  // Verifica autenticação no carregamento inicial
  const isAuthenticated = localStorage.getItem('authToken')
  if (!isAuthenticated && router.currentRoute.value.path !== '/login') {
    router.push('/login')
  }
})
</script>

<template>
  <v-app>
    <router-view></router-view>
  </v-app>
</template>
