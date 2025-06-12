<script setup>
import { ref, onMounted } from 'vue'

const doctores = ref([])
const error = ref('')

async function fetchDoctores() {
  try {
    const res = await fetch('http://localhost/doctores', {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
    })
    if (!res.ok) {
      throw new Error('Error al obtener los doctores')
    }
    const data = await res.json()
    console.log('Doctores recibidos:', data)
    console.log(data)
    doctores.value = data.medicos
    } catch (err) {
    console.error('Error al obtener los doctores:', err)
    error.value = 'Error al obtener los doctores'
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchDoctores()
})

</script>

<template>

<!-- card doctores -->
<div>
  <h1>Doctores</h1>
  <div v-if="error" style="color: red">{{ error }}</div>
  <div v-if="!error && doctores.length === 0">No hay doctores disponibles</div>
  <ul v-if="doctores.length > 0">
    <li v-for="doctor in doctores" :key="doctor.id">
      <div class="card">
        <h2>{{ doctor.nombre }}</h2>
        <p>Email: {{ doctor.email }}</p>
        <p>Especialidad: {{ doctor.especialidad }}</p>
      </div>
    </li>
  </ul>
</div>

</template>