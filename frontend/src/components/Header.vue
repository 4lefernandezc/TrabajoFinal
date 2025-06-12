<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const pestañas = ref([
  { nombre: 'Inicio', ruta: '/', oculto: false },
  { nombre: 'Agenda', ruta: '/agenda', oculto: false },
  { nombre: 'Horarios', ruta: '/agenda', oculto: true }, // Oculto por defecto
  { nombre: 'Pacientes', ruta: '/pacientes', oculto: false },
  { nombre: 'Doctores', ruta: '/doctores', oculto: false },
  { nombre: 'Especialidades', ruta: '/especialidades', oculto: false },
  { nombre: 'Reservas', ruta: '/citas', oculto: true },
  // { nombre: 'Notificaciones', ruta: '/notificaciones' }
])

const rol = computed(() => localStorage.getItem('role') || 'user')

const pestañaActiva = ref('/')
const cambiarPestaña = (ruta) => {
  pestañaActiva.value = ruta
  router.push(ruta)
}

// Recibe el token como prop
const props = defineProps({
  token: String
})

// Emite el evento logout al padre (App.vue)
const emit = defineEmits(['logout'])
function handleLogout() {
  emit('logout')
}
</script>

<template>
  <header>
    <nav>
      <ul>
        <li v-for="pestaña in pestañas" :key="pestaña.ruta">
          <a v-if="pestaña.oculto || rol === 'admin' || rol === 'Medico'"
            href="#"
            :class="{ 'active': pestañaActiva === pestaña.ruta }"
            @click.prevent="cambiarPestaña(pestaña.ruta)"
          >
            {{ pestaña.nombre }}
          </a>
        </li>
      </ul>
      <!-- Botón de logout solo si hay token -->
      <button v-if="token" @click="handleLogout" style="margin-left:2em">
        Cerrar sesión
      </button>
    </nav>
  </header>
</template>

<style scoped>

header {
  background-color: #333;
  color: white;
  padding: 1em;
}
nav ul {
  list-style: none;
  padding: 0;
  display: flex;
}
nav li {
  margin-right: 1em;
}
nav a {
  color: white;
  text-decoration: none;
}
nav a.active {
  font-weight: bold;
  text-decoration: underline;
}
nav button {
  background-color: #f00;
  color: white;
  border: none;
  padding: 0.5em 1em;
  cursor: pointer;
}
nav button:hover {
  background-color: #c00;
}
nav button:focus {
  outline: none;
}
nav button:active {
  background-color: #900;
}
nav button:disabled {
  background-color: #666;
  cursor: not-allowed;
}
nav button:disabled:hover {
  background-color: #666;
}
nav button:disabled:focus {
  outline: none;
}

nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

</style>