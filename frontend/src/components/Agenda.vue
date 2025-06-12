<script setup>
import { ref, onMounted } from 'vue'

const appointments = ref([])
const schedules = ref([])
const loading = ref(false)
const error = ref('')
const user = localStorage.getItem('email');
const newAppointment = ref({
  doctorId: '',
  patientId: '',
  specialtyId: '',
  appointmentDate: '',
  startTime: '',
  endTime: '',
  notes: ''
})
const token = localStorage.getItem('token')
const role = localStorage.getItem('role')

const especialidades = ref([])
const newSchedule = ref({
  doctorId: '',
  specialtyIds: [],
  weeklySchedule: [
    {
      dayOfWeek: 1,
      timeSlots: [
        { startTime: '', endTime: '' }
      ],
      isWorkingDay: true
    }
  ]
})

// Función para hacer peticiones GraphQL
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

async function cargarSchedules() {
  const query = `
    query {
      doctorSchedules {
        id
        doctorId
        specialties { id name }
        weeklySchedule { dayOfWeek timeSlots { startTime endTime isAvailable } isWorkingDay }
        isActive
        createdAt
        updatedAt
      }
    }
  `
  const data = await gqlRequest(query)
  if (data) schedules.value = data.doctorSchedules
}

// Listar citas
async function loadAppointments() {
  const query = `
    query {
      appointments {
        id
        doctorId
        patientId
        specialty { id name }
        appointmentDate
        startTime
        endTime
        status
        notes
      }
    }
  `
  const data = await gqlRequest(query)
  if (data) appointments.value = data.appointments
}

// Crear cita
// async function createAppointment() {
//   const mutation = `
//     mutation($input: AppointmentInput!) {
//       createAppointment(input: $input) {
//         id
//         doctorId
//         patientId
//         specialty { id name }
//         appointmentDate
//         startTime
//         endTime
//         status
//         notes
//       }
//     }
//   `
//   const data = await gqlRequest(mutation, { input: newAppointment.value })
//   if (data) {
//     appointments.value.push(data.createAppointment)
//     Object.keys(newAppointment.value).forEach(k => newAppointment.value[k] = '')
//   }
// }

// Actualizar estado de cita
async function updateStatus(id, status) {
  const mutation = `
    mutation($id: ID!, $status: String!) {
      updateAppointmentStatus(id: $id, status: $status) {
        id
        status
      }
    }
  `
  const data = await gqlRequest(mutation, { id, status })
  if (data) {
    const apt = appointments.value.find(a => a.id === id)
    if (apt) apt.status = data.updateAppointmentStatus.status
  }
}

// Cancelar cita
async function cancelAppointment(id) {
  const reason = prompt('Motivo de la cancelación:')
  if (!reason) return
  const mutation = `
    mutation($id: ID!, $reason: String) {
      cancelAppointment(id: $id, reason: $reason) {
        id
        status
        notes
      }
    }
  `
  const data = await gqlRequest(mutation, { id, reason })
  if (data) {
    const apt = appointments.value.find(a => a.id === id)
    if (apt) {
      apt.status = data.cancelAppointment.status
      apt.notes = data.cancelAppointment.notes
    }
  }
}

async function createSchedule() {
  // Asegúrate de que specialtyIds sea un array (aunque sea de un solo elemento)
  if (typeof newSchedule.value.specialtyIds === 'string') {
    newSchedule.value.specialtyIds = [newSchedule.value.specialtyIds]
  }
  // Actualiza el primer elemento del weeklySchedule con los datos del formulario
  newSchedule.value.weeklySchedule[0] = {
    dayOfWeek: parseInt(newSchedule.value.weeklySchedule[0].dayOfWeek),
    timeSlots: [
      {
        startTime: newSchedule.value.weeklySchedule[0].timeSlots[0].startTime,
        endTime: newSchedule.value.weeklySchedule[0].timeSlots[0].endTime
      }
    ],
    isWorkingDay: true
  }
  const mutation = `
    mutation($input: DoctorScheduleInput!) {
      createDoctorSchedule(input: $input) {
        id
        doctorId
        specialties { id name }
        weeklySchedule { dayOfWeek timeSlots { startTime endTime isAvailable } isWorkingDay }
        isActive
        createdAt
        updatedAt
      }
    }
  `
  const data = await gqlRequest(mutation, { input: newSchedule.value })
  if (data) {
    // Puedes recargar agendas o mostrar mensaje de éxito aquí
    Object.assign(newSchedule.value, {
      doctorId: '',
      specialtyIds: [],
      weeklySchedule: [
        {
          dayOfWeek: 1,
          timeSlots: [{ startTime: '', endTime: '' }],
          isWorkingDay: true
        }
      ]
    })
  }
}

onMounted(() => {
  document.title = 'Agenda App - Horarios'
  loadAppointments()
  console.log('Token JWT:', token)
  console.log('Rol del usuario:', role)
  cargarEspecialidades()
  cargarSchedules()
})
</script>

<template>
  <div>
    <h2>HORARIOS</h2>
    <div v-if="error" style="color:red">{{ error }}</div>
    <div v-if="loading">Cargando...</div>

    <!-- Crear nueva cita -->
    <!-- <form @submit.prevent="createAppointment" style="margin-bottom:2em">
      <h3>Nueva cita</h3>
      <input v-model="newAppointment.doctorId" placeholder="Doctor ID" required />
      <input v-model="newAppointment.patientId" placeholder="Paciente ID" required />
      <input v-model="newAppointment.specialtyId" placeholder="Especialidad ID" required />
      <input v-model="newAppointment.appointmentDate" type="date" required />
      <input v-model="newAppointment.startTime" placeholder="Hora inicio (HH:mm)" required />
      <input v-model="newAppointment.endTime" placeholder="Hora fin (HH:mm)" required />
      <input v-model="newAppointment.notes" placeholder="Notas" />
      <button type="submit">Crear cita</button>
    </form> -->

    <!-- Crear nuevo horario -->
    <form @submit.prevent="createSchedule" style="margin-bottom:2em">
  <h3>Nuevo horario</h3>
  <input v-model="newSchedule.doctorId" :placeholder="user" required />
  <select v-model="newSchedule.specialtyIds" placeholder="Especialidad" required>
    <option value="" disabled selected>Especialidad</option>
    <option v-for="spec in especialidades" :key="spec.id" :value="spec.id">{{ spec.name }}</option>
  </select>
  <select v-model="newSchedule.weeklySchedule[0].dayOfWeek" required>
    <option value="" disabled selected>Día de la semana</option>
    <option :value="1">Lunes</option>
    <option :value="2">Martes</option>
    <option :value="3">Miércoles</option>
    <option :value="4">Jueves</option>
    <option :value="5">Viernes</option>
    <option :value="6">Sábado</option>
    <option :value="7">Domingo</option>
  </select>
  <input v-model="newSchedule.weeklySchedule[0].timeSlots[0].startTime" placeholder="Hora inicio (HH:mm)" required />
  <input v-model="newSchedule.weeklySchedule[0].timeSlots[0].endTime" placeholder="Hora fin (HH:mm)" required />
  <button type="submit">Crear horario</button>
</form>

    <!-- schedules disponibles -->
    <h3>Horarios disponibles</h3>
    <table border="1" cellpadding="5">
      <thead>
        <tr>
          <th>Doctor ID</th>
          <th>Especialidades</th>
          <th>Día de la semana</th>
          <th>Horario</th>
          <th>Activo</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="schedule in schedules" :key="schedule.id">
          <td>{{ schedule.doctorId }}</td>
          <td>{{ schedule.specialties.map(s => s.name).join(', ') }}</td>
          <td>{{ schedule.weeklySchedule[0].dayOfWeek }}</td>
          <td>{{ schedule.weeklySchedule[0].timeSlots[0].startTime }} - {{ schedule.weeklySchedule[0].timeSlots[0].endTime }}</td>
          <td>{{ schedule.isActive ? 'Sí' : 'No' }}</td>
          <td>
            <!-- Aquí puedes agregar botones para editar o eliminar el horario -->
            <!-- Por ejemplo: -->
            <!-- <button @click="editSchedule(schedule.id)">Editar</button> -->
            <!-- <button @click="deleteSchedule(schedule.id)">Eliminar</button> -->
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>  

h2 {
  color: #333;
  font-size: 1.5em;
  margin-bottom: 0.5em;
}
p {
  color: #666;
  font-size: 1.2em;
  margin-bottom: 1em;
}
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1em;
}
th, td {
  padding: 0.5em;
  text-align: left;
  border-bottom: 1px solid #ddd;
}
th {
  background-color: #f2f2f2;
  font-weight: bold;
}
button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 0.5em 1em;
  cursor: pointer;
}
button:hover {
  background-color: #0056b3;
}
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
button:disabled:hover {
  background-color: #ccc;
}

button:focus {
  outline: none;
}
button:active {
  background-color: #0056b3;
}
input, select {
  padding: 0.5em;
  margin: 0.5em 0;
  width: calc(100% - 1em);
  box-sizing: border-box;
}
input[type="date"],
input[type="time"] {
  width: auto;
}
input[type="date"] {
  padding: 0.5em;
  width: 150px;
}
input[type="time"] {
  width: 100px;
}
input[type="text"],
input[type="number"],
input[type="email"],
input[type="password"],
input[type="search"],
input[type="tel"],
input[type="url"],
input[type="datetime-local"],
input[type="month"],
input[type="week"],
input[type="time"] {
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}
input:focus,
select:focus {
  border-color: #007bff;
  outline: none;
}


</style>