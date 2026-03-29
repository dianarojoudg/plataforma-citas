<template>
  <section class="list-section">
    <h2>📋 Lista de Pacientes Registrados</h2>
    
    <Buscador @buscar="filtrarCitas" />
    
    <TablaPacientes 
      :citas="citasFiltradas" 
      @eliminar="eliminarCita"
    />
    
    <Estadisticas :citas="citas" />
  </section>
</template>

<script>
export default {
  name: 'AgendaView',
  props: {
    citas: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      filtro: ''
    }
  },
  computed: {
    // Búsqueda filtrada
    citasFiltradas() {
      return this.citas.filter(cita => 
        cita.nombre.toLowerCase().includes(this.filtro.toLowerCase()) ||
        cita.telefono.includes(this.filtro)
      )
    }
  },
  methods: {
    filtrarCitas(termino) {
      this.filtro = termino
    },
    
    eliminarCita(id) {
      if(confirm('¿Estás seguro de eliminar esta cita?')) {
        const citas = JSON.parse(localStorage.getItem('citas')) || []
        const citasActualizadas = citas.filter(cita => cita.id !== id)
        localStorage.setItem('citas', JSON.stringify(citasActualizadas))
        
        this.mostrarNotificacion('Cita eliminada', 'success')
        this.$emit('cita-eliminada')
      }
    },
    
    mostrarNotificacion(mensaje, tipo) {
      const notification = document.createElement('div')
      notification.className = `notification ${tipo}`
      notification.textContent = mensaje
      document.body.appendChild(notification)
      
      setTimeout(() => {
        notification.remove()
      }, 3000)
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
  transition: all 0.3s ease;
}

.agenda-view h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 1.5em;
  border-bottom: 2px solid #667eea;
  padding-bottom: 10px;
}

.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 15px 20px;
  border-radius: 5px;
  color: white;
  font-weight: 500;
  animation: slideIn 0.3s ease;
  z-index: 1000;
}

.notification.success {
  background: linear-gradient(135deg, #84fab0 0%, #8fd3f4 100%);
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}
</style>