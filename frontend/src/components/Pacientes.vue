<script setup>
import { ref, onMounted } from 'vue'

const pacientes = ref([])
const error = ref('')
const loading = ref(true)
const token = localStorage.getItem('token')
const role = localStorage.getItem('role')

async function fetchPacientes() {
  try {
    const res = await fetch('http://localhost/pacientes', {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    })
    if (!res.ok) {
      throw new Error('Error al obtener los pacientes')
    }
    const data = await res.json()
    console.log('Pacientes recibidos:', data)
    pacientes.value = data
    } catch (err) {
    console.error('Error al obtener los pacientes:', err)
    error.value = 'Error al obtener los pacientes'
    } finally {
    loading.value = false
    }
}

onMounted(() => {
  fetchPacientes()
})

</script>

<template>
  <div>
    <h1>Pacientes</h1>
    <div v-if="loading">Cargando pacientes...</div>
    <div v-if="error" style="color: red">{{ error }}</div>
    <ul v-if="!loading && !error">
      <li v-for="paciente in pacientes" :key="paciente.id">
        {{ paciente.nombre }} - {{ paciente.email }}
      </li>
    </ul>
  </div>
    <div v-if="role === 'admin'">
        <h2>Acciones de administrador</h2>
        <button @click="fetchPacientes">Refrescar Pacientes</button>
        <button @click="fetchPacientes">Agregar Paciente</button>
        <button @click="fetchPacientes">Eliminar Paciente</button>
        <button @click="fetchPacientes">Editar Paciente</button>
    </div>
</template>
<style scoped>

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
    h2 {
        color: #555;
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


</style>