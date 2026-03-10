<template>
  <div class="q-pa-md" style="max-width: 400px; margin:auto; margin-top:100px;">
    <q-input dense outlined v-model="username" label="Username" />
    <q-input dense outlined v-model="password" label="Password" type="password" class="q-mt-md" />

    <q-btn dense color="primary" label="Login" class="q-mt-lg" @click="loginUser" />
  </div>
</template>

<script setup>
import { ref } from 'vue'

const username = ref('')
const password = ref('')

const loginUser = () => {
  console.log('Login', username.value, password.value)
}
</script>