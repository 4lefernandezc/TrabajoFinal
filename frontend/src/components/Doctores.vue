<script setup>
import { ref, onMounted } from 'vue'

const doctores = ref([])
const error = ref('')
const loading = ref(true)
const token = localStorage.getItem('token')

// Formulario de registro
const email = ref('')
const password = ref('')
const especialidad = ref('')
const regLoading = ref(false)
const regError = ref('')
const regSuccess = ref('')

// Modal
const showModal = ref(false)

async function fetchDoctores() {
  try {
    const res = await fetch('http://localhost/doctores', {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    })
    if (!res.ok) {
      throw new Error('Error al obtener los doctores')
    }
    const data = await res.json()
    doctores.value = data.medicos || []
    console.log('Doctores obtenidos:', doctores.value)
  } catch (err) {
    error.value = 'Error al obtener los doctores'
  } finally {
    loading.value = false
  }
}

async function registrarDoctor() {
  regError.value = ''
  regSuccess.value = ''
  regLoading.value = true
  try {
    const res = await fetch('http://localhost/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        email: email.value,
        password: password.value,
        role: 'Medico',
        especialidad: especialidad.value
      })
    })
    const data = await res.json()
    if (!res.ok) {
      throw new Error(data.message || 'Error al registrar doctor')
    }
    regSuccess.value = 'Doctor registrado exitosamente'
    email.value = ''
    password.value = ''
    especialidad.value = ''
    fetchDoctores()
    showModal.value = false
  } catch (err) {
    regError.value = err.message
  } finally {
    regLoading.value = false
  }
}

function openModal() {
  regError.value = ''
  regSuccess.value = ''
  email.value = ''
  password.value = ''
  especialidad.value = ''
  showModal.value = true
}

function closeModal() {
  showModal.value = false
}

onMounted(() => {
  fetchDoctores()
})
</script>

<template>
  <div>
    <h1>Doctores</h1>
    <div v-if="loading">Cargando doctores...</div>
    <div v-if="error" style="color: red">{{ error }}</div>
    <template v-if="!error">
      <div v-if="!doctores || doctores.length === 0" style="color: #888;">
        No hay doctores registrados.
      </div>
      <div v-else class="cards-container">
        <div v-for="doctor in doctores" :key="doctor.id" class="paciente-card">
          <div class="paciente-avatar">
            <span>{{ doctor.nombre ?doctor.nombre.charAt(0).toUpperCase() : doctor.email.charAt(0).toUpperCase()
              }}</span>
          </div>
          <div class="paciente-info">
            <div class="paciente-id">ID: {{ doctor.id }}</div>
            <div class="paciente-email">{{ doctor.email }}</div>
            <div class="paciente-especialidad">Especialidad: {{ doctor.especialidad }}</div>
          </div>
        </div>
      </div>
    </template>
    <button @click="openModal" style="margin-top:2em;">Registrar Doctor</button>
  </div>
  <!-- Modal -->
  <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
    <div class="modal-content">
      <button class="modal-close" @click="closeModal">&times;</button>
      <h3>Registrar Doctor</h3>
      <form @submit.prevent="registrarDoctor">
        <div>
          <label>Email:</label>
          <input v-model="email" type="email" required />
        </div>
        <div>
          <label>Contraseña:</label>
          <input v-model="password" type="password" required />
        </div>
        <div>
          <label>Especialidad:</label>
          <input v-model="especialidad" type="text" required />
        </div>
        <button type="submit" :disabled="regLoading">Registrar</button>
      </form>
      <div v-if="regLoading">Registrando...</div>
      <div v-if="regError" style="color: red">{{ regError }}</div>
      <div v-if="regSuccess" style="color: green">{{ regSuccess }}</div>
    </div>
  </div>
</template>

<style scoped>
.cards-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5em;
  margin-top: 2em;
}

.paciente-card {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  padding: 1.5em 1.2em;
  min-width: 220px;
  max-width: 260px;
  display: flex;
  align-items: center;
  transition: box-shadow 0.2s;
  border: 1px solid #f0f0f0;
}

.paciente-card:hover {
  box-shadow: 0 4px 20px rgba(0, 123, 255, 0.15);
  border-color: #007bff33;
}

.paciente-avatar {
  width: 48px;
  height: 48px;
  background: #007bff22;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.7em;
  color: #007bff;
  font-weight: bold;
  margin-right: 1em;
  flex-shrink: 0;
}

.paciente-info {
  display: flex;
  flex-direction: column;
  gap: 0.3em;
}

.paciente-id {
  font-size: 0.95em;
  color: #888;
}

.paciente-email {
  font-size: 1.1em;
  color: #222;
  font-weight: 500;
}

.paciente-especialidad {
  font-size: 1em;
  color: #555;
}

/* ...resto de estilos igual... */
ul {
  list-style-type: none;
  padding: 0;
}

li {
  padding: 0.5em;
  border-bottom: 1px solid #ccc;
}

li:last-child {
  border-bottom: none;
  color: white;
  text-decoration: none;
}

li:hover {
  background-color: #f0f0f0;
}

h1 {
  color: #333;
}

button {
  margin: 0.5em;
  padding: 0.5em 1em;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:disabled {
  background-color: #aaa;
}

/* Modal styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: #fff;
  padding: 2em;
  border-radius: 8px;
  min-width: 320px;
  max-width: 90vw;
  position: relative;
  box-shadow: 0 2px 16px rgba(0, 0, 0, 0.2);
}

.modal-close {
  position: absolute;
  top: 0.5em;
  right: 0.7em;
  background: none;
  border: none;
  font-size: 2em;
  color: #888;
  cursor: pointer;
}

form {
  max-width: 400px;
  margin: 1em auto 0 auto;
  padding: 1em 0 0 0;
  border: none;
  border-radius: 8px;
  background: none;
}

label {
  display: block;
  margin-bottom: 0.5em;
  color: #333;
}

input {
  width: 100%;
  padding: 0.5em;
  margin-bottom: 1em;
  border: 1px solid #ccc;
  border-radius: 4px;
}
</style>