<template>
  <div class="agenda-view">
    <h2>📋 Lista de Citas</h2>
    <Buscador @buscar="filtrarCitas" />
    <TablaPacientes 
      :citas="citasFiltradas" 
      @eliminar="eliminarCita"
      @editar="editarCita"
    />
    <Estadisticas :citas="citas" />
  </div>
</template>

<script>
import Buscador from '../components/Buscador.vue'
import TablaPacientes from '../components/TablaPacientes.vue'
import Estadisticas from '../components/Estadisticas.vue'

export default {
  name: 'AgendaView',
  components: {
    Buscador,
    TablaPacientes,
    Estadisticas
  },
  props: {
    citas: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      terminoBusqueda: ''
    }
  },
  computed: {
    citasFiltradas() {
      if (!this.terminoBusqueda) return this.citas
      
      const busqueda = this.terminoBusqueda.toLowerCase()
      return this.citas.filter(cita => 
        cita.nombre.toLowerCase().includes(busqueda) ||
        cita.telefono.includes(busqueda)
      )
    }
  },
  methods: {
    eliminarCita(id) {
      const citas = JSON.parse(localStorage.getItem('citas')) || []
      const citasActualizadas = citas.filter(cita => cita.id !== id)
      localStorage.setItem('citas', JSON.stringify(citasActualizadas))
      this.$emit('cita-eliminada')
    },
    
    editarCita(cita) {
      this.$emit('cita-editar', cita)
    },
    
    filtrarCitas(termino) {
      this.terminoBusqueda = termino
    }
  }
}
</script>

<style scoped>
.agenda-view {
  background: white;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.agenda-view h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 1.5em;
  border-bottom: 2px solid #667eea;
  padding-bottom: 10px;
}
</style>