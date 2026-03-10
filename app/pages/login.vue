<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <!-- Modern Gradient Background -->
      <q-page class="flex flex-center login-bg">
        <q-card class="login-card shadow-24 overflow-hidden">
          <div class="row full-height">
            
            <!-- Left Side: Visual/Brand (Optional pan Professional disto) -->
            <div class="col-12 col-md-5 bg-primary flex flex-center q-pa-xl text-white">
              <div class="text-center">
                <q-icon name="rocket_launch" size="80px" class="q-mb-md animate-bounce" />
                <div class="text-h4 text-weight-bolder">LeadGen</div>
                <div class="text-subtitle1 q-mt-sm text-grey-4">USA Based Lead Generation System</div>
              </div>
            </div>

            <!-- Right Side: Login Form -->
            <div class="col-12 col-md-7 q-pa-xl bg-white flex flex-center">
              <div style="width: 100%; max-width: 320px;">
                <div class="text-h5 text-weight-bold text-grey-9 q-mb-xs">Welcome Back</div>
                <div class="text-caption text-grey-6 q-mb-xl">Please login to your admin account</div>

                <q-form @submit="loginUser" class="q-gutter-y-md">
                  <!-- Username Input -->
                  <q-input 
                    v-model="username" 
                    label="Username" 
                    outlined 
                    dense
                    color="primary"
                    bg-color="grey-1"
                  >
                    <template v-slot:prepend>
                      <q-icon name="person_outline" color="primary" />
                    </template>
                  </q-input>

                  <!-- Password Input -->
                  <q-input
                    v-model="password"
                    :type="isPwd ? 'password' : 'text'"
                    label="Password"
                    outlined
                    dense
                    color="primary"
                    bg-color="grey-1"
                  >
                    <template v-slot:prepend>
                      <q-icon name="lock_open" color="primary" />
                    </template>
                    <template v-slot:append>
                      <q-icon
                        :name="isPwd ? 'visibility_off' : 'visibility'"
                        class="cursor-pointer"
                        @click="isPwd = !isPwd"
                      />
                    </template>
                  </q-input>

                  <!-- Login Button -->
                  <q-btn
                    type="submit"
                    label="Login to Dashboard"
                    color="primary"
                    unelevated
                    class="full-width q-py-sm text-weight-bold br-8"
                    size="md"
                    :loading="loading"
                  />

                  <!-- Error Message -->
                  <q-banner v-if="error" dense rounded class="bg-red-1 text-negative q-mt-md">
                    <template v-slot:avatar>
                      <q-icon name="error" color="negative" />
                    </template>
                    {{ error }}
                  </q-banner>
                </q-form>

                <div class="text-center q-mt-xl">
                  <div class="text-caption text-grey-5">© 2024 LeadGen Inc. All Rights Reserved.</div>
                </div>
              </div>
            </div>
          </div>
        </q-card>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const router = useRouter()
const username = ref('')
const password = ref('')
const error = ref('')
const loading = ref(false)
const isPwd = ref(true)

const loginUser = async () => {
  if (!username.value || !password.value) {
    error.value = 'Please enter both fields'
    return
  }
  
  error.value = ''
  loading.value = true
  
  try {
    const response = await axios.post(
      'http://localhost:8000/api/login/',
      { username: username.value, password: password.value },
      { withCredentials: true }
    )

    localStorage.setItem('isLoggedIn', 'true')
    router.push('/dashboard')
  } catch (e) {
    error.value = 'Invalid username or password'
    console.error("Login Error:", e)
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
/* Gradient Background */
.login-bg {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

/* Card Styling */
.login-card {
  width: 900px;
  max-width: 95vw;
  min-height: 550px;
  border-radius: 24px;
}

/* Rounded Buttons/Inputs */
.br-8 {
  border-radius: 8px;
}

/* Input Overrides */
:deep(.q-field--outlined .q-field__control) {
  border-radius: 10px;
}

/* Animation */
.animate-bounce {
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}

/* Responsiveness */
@media (max-width: 768px) {
  .login-card {
    width: 400px;
  }
  .bg-primary {
    display: none; /* Mobile var left side kadhun taka */
  }
}
</style>
