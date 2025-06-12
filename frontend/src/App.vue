<script setup>
import Header from './components/Header.vue'
import Footer from './components/Footer.vue'
import { useRouter } from 'vue-router'
import { ref, watchEffect } from 'vue'

const router = useRouter()
const token = ref(localStorage.getItem('token'))

// Mantén el token sincronizado con localStorage
watchEffect(() => {
  if (token.value) {
    localStorage.setItem('token', token.value)
  } else {
    localStorage.removeItem('token')
  }
})

// Escucha eventos de login/logout si los emites desde Login.vue
function onLogin(newToken) {
  token.value = newToken
  router.push('/agenda')
}
function logout() {
  token.value = null
  router.push('/login')
}
</script>

<template>
  <div>
    <Header :token="token" @logout="logout" />
    <router-view @login="onLogin" :token="token" :user />
    <Footer />
  </div>
</template>