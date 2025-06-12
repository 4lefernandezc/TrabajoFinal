<script setup>
import { ref, onMounted } from 'vue'

const especialidades = ref([])
const loading = ref(false)
const error = ref('')
const nuevaEspecialidad = ref({
  name: '',
  description: ''
})
const token = localStorage.getItem('token')

// Función para peticiones GraphQL
async function gqlRequest(query, variables = {}) {
  loading.value = true
  error.value = ''
  try {
    const res = await fetch('http://localhost/agenda/graphql', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...(token ? { Authorization: `Bearer ${token}` } : {})
      },
      body: JSON.stringify({ query, variables })
    })
    const data = await res.json()
    if (data.errors) throw new Error(data.errors[0].message)
    return data.data
  } catch (err) {
    error.value = err.message
    return null
  } finally {
    loading.value = false
  }
}

// Listar especialidades
async function cargarEspecialidades() {
  const query = `
    query {
      specialties {
        id
        name
        description
        isActive
        createdAt
        updatedAt
      }
    }
  `
  const data = await gqlRequest(query)
  if (data) especialidades.value = data.specialties
}

// Crear especialidad
async function crearEspecialidad() {
  const mutation = `
    mutation($input: SpecialtyInput!) {
      createSpecialty(input: $input) {
        id
        name
        description
        isActive
      }
    }
  `
  const data = await gqlRequest(mutation, { input: nuevaEspecialidad.value })
  if (data) {
    especialidades.value.push(data.createSpecialty)
    nuevaEspecialidad.value.name = ''
    nuevaEspecialidad.value.description = ''
  }
}

// Eliminar especialidad
async function eliminarEspecialidad(id) {
  if (!confirm('¿Eliminar esta especialidad?')) return
  const mutation = `
    mutation($id: ID!) {
      deleteSpecialty(id: $id)
    }
  `
  const data = await gqlRequest(mutation, { id })
  if (data && data.deleteSpecialty) {
    especialidades.value = especialidades.value.filter(e => e.id !== id)
  }
}

onMounted(cargarEspecialidades)
</script>

<template>
  <div>
    <h2>Gestión de Especialidades</h2>
    <div v-if="error" style="color:red">{{ error }}</div>
    <div v-if="loading">Cargando...</div>

    <!-- Crear nueva especialidad -->
    <form @submit.prevent="crearEspecialidad" style="margin-bottom:2em">
      <h3>Nueva especialidad</h3>
      <input v-model="nuevaEspecialidad.name" placeholder="Nombre" required />
      <textarea v-model="nuevaEspecialidad.description" placeholder="Descripción" required />
      <button type="submit">Crear</button>
    </form>

    <!-- Listado de especialidades -->
    <table border="1" cellpadding="5">
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Descripción</th>
          <th>Activo</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="esp in especialidades" :key="esp.id">
          <td>{{ esp.name }}</td>
          <td>{{ esp.description }}</td>
          <td>{{ esp.isActive ? 'Sí' : 'No' }}</td>
          <td>
            <button @click="eliminarEspecialidad(esp.id)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1em;
}
th, td {
  padding: 8px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}
th {
  background-color: #f2f2f2;
}
td button {
  background-color: #f44336;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
td button:hover {
  background-color: #d32f2f;
}
td input {
  width: 100%;
  padding: 5px;
  box-sizing: border-box;
}
td input[type="text"] {
  width: auto;
  margin-right: 10px;
}
td input[type="text"]:last-child {
  margin-right: 0;
}

td input[type="text"]:focus {
  outline: none;
  border: 1px solid #007bff;
}
td input[type="text"]:disabled {
  background-color: #f0f0f0;
  cursor: not-allowed;
}
td input[type="text"]:disabled:hover {
  background-color: #f0f0f0;
}
td input[type="text"]:disabled:focus {
  outline: none;
}

td button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
td button:disabled:hover {
  background-color: #ccc;
}

input, button {
  font-size: 14px;
  border-radius: 4px;
}
input {
  border: 1px solid #ccc;
  padding: 8px;
  width: calc(100% - 16px);
  box-sizing: border-box;
}
input:focus {
  border-color: #007bff;
  outline: none;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 8px 16px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
button:focus {
  outline: none;
}
button:active {
  background-color: #004494;
}
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
button:disabled:hover {
  background-color: #ccc;
}
button:disabled:focus {
  outline: none;
}
textarea {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 4px;
}


</style>