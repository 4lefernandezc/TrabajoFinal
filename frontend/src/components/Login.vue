<script setup>
import { ref } from 'vue'

const emit = defineEmits(['login'])
const email = ref('')
const password = ref('')
const error = ref('')
const token = ref('')

async function handleLogin() {
  error.value = ''
  token.value = ''
  const res = await fetch('http://localhost/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email: email.value, password: password.value })
  })
  const data = await res.json()
  console.log('Respuesta del servidor:', data)
  if (data.token) {
    token.value = data.token
    emit('login', data.token)
  } else {
    error.value = data.message || 'Error al iniciar sesión'
  }
  if (data.token) {
    console.log(data.token)
    console.log('Token recibido:', data.token)
  localStorage.setItem('token', data.token)
  emit('login', data.token)
  // dentro del token esta informacion como el rol del usuario lo guardamos en localStorage
  localStorage.setItem('role', data.user.role)
  localStorage.setItem('email', data.user.email)
  } else {
    localStorage.removeItem('token')
    localStorage.removeItem('role')
  }
  // Redirige al home si el login es exitoso
  if (data.token) {


  window.location.href = '/' // Redirige al home
}
}
</script>

<template>
  <form @submit.prevent="handleLogin">
    <h2>Login</h2>
    <input v-model="email" placeholder="Email" required />
    <input v-model="password" type="password" placeholder="Password" required />
    <button type="submit">Entrar</button>
    <div v-if="error" style="color:red">{{ error }}</div>
    <div v-if="token" style="word-break:break-all">
      <strong>Token JWT:</strong>
      <div>{{ token }}</div>
    </div>
  </form>
</template>

<style scoped>
form {
  max-width: 300px;
  margin: 100px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
input {
  width: 92%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
}
button {
  width: 100%;
  padding: 10px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
button:hover {
  background-color: #218838;
}
div {
  margin-top: 10px;
}
div strong {
  display: block;
  margin-bottom: 5px;
}
div {
  word-break: break-all;
}
h2 {
  text-align: center;
  margin-bottom: 20px;
}
</style>
