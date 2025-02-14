//src/views/LoginView.vue
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

// Estado do formulário
const form = ref({
  email: '',
  password: '',
  rememberMe: false
})

// Estado de carregamento e erro
const loading = ref(false)
const error = ref('')
const showPassword = ref(false)

// Função para fazer login
const handleLogin = async () => {
  loading.value = true
  error.value = ''

  try {
    // Simula uma validação básica
    if (!form.value.email || !form.value.password) {
      throw new Error('Por favor, preencha todos os campos')
    }

    // Aqui você pode adicionar sua lógica de autenticação real
    // Por enquanto, vamos simular um login com credenciais fixas
// Em LoginView.vue, na função handleLogin
if (form.value.email === 'admin@example.com' && form.value.password === 'admin123') {
  await new Promise(resolve => setTimeout(resolve, 1000))

  localStorage.setItem('authToken', 'dummy-token')
  localStorage.setItem('user', JSON.stringify({
    name: 'Administrador',
    email: form.value.email
  }))

  // Redireciona para /dashboard em vez de /home
  router.push('/dashboard')
} else {
  throw new Error('Credenciais inválidas')
}
  } catch (err: any) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <v-container fluid class="fill-height">
    <v-row align="center" justify="center">
      <v-col cols="12" sm="8" md="4">
        <v-card class="elevation-12">
          <!-- Cabeçalho -->
          <v-card-title class="text-h5 text-center py-4 bg-primary text-white">
            <v-icon size="32" class="mr-2">mdi-chart-box</v-icon>
            Data Company Sistemas
          </v-card-title>

          <!-- Formulário -->
          <v-card-text class="pa-6">
            <v-form @submit.prevent="handleLogin">
              <!-- Email -->
              <v-text-field
                v-model="form.email"
                label="Email"
                type="email"
                prepend-inner-icon="mdi-email"
                variant="outlined"
                :error-messages="error ? [error] : []"
                required
              ></v-text-field>

              <!-- Senha -->
              <v-text-field
                v-model="form.password"
                label="Senha"
                :type="showPassword ? 'text' : 'password'"
                prepend-inner-icon="mdi-lock"
                :append-inner-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                @click:append-inner="showPassword = !showPassword"
                variant="outlined"
                required
              ></v-text-field>

              <!-- Lembrar-me -->
              <v-checkbox
                v-model="form.rememberMe"
                label="Lembrar-me"
                color="primary"
                hide-details
                class="mb-4"
              ></v-checkbox>

              <!-- Botão de Login -->
              <v-btn
                type="submit"
                color="primary"
                block
                :loading="loading"
                size="large"
                class="mt-2"
              >
                Entrar
              </v-btn>
            </v-form>
          </v-card-text>

          <!-- Links extras -->
          <v-card-actions class="justify-center pb-6">
            <v-btn variant="text" color="primary">
              Esqueceu a senha?
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
.v-container {
  background: linear-gradient(135deg, #1867C0 0%, #5CBBF6 100%);
}

.v-card {
  border-radius: 12px;
}

.v-card-title {
  border-top-left-radius: 12px;
  border-top-right-radius: 12px;
}
</style>
