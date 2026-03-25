<template>
  <div class="container">
    <header>
      <h1>🏥 Consultorio Médico</h1>
      <p>Sistema de Registro y Visualización de Pacientes</p>
    </header>
    
    <main>
      <FormularioCita 
        :citaParaEditar="citaEnEdicion"
        @cita-agregada="cargarCitas"
        @cita-actualizada="cargarCitas"
        @cancelar-edicion="cancelarEdicion"
      />
      <AgendaView 
        :citas="citas" 
        @cita-eliminada="cargarCitas"
        @cita-editar="editarCita"
      />
    </main>
  </div>
</template>

<script>
import FormularioCita from './components/FormularioCita.vue'
import AgendaView from './views/AgendaView.vue'

export default {
  name: 'App',
  components: {
    FormularioCita,
    AgendaView
  },
  data() {
    return {
      citas: [],
      citaEnEdicion: null
    }
  },
  methods: {
    cargarCitas() {
      this.citas = JSON.parse(localStorage.getItem('citas')) || []
      this.citaEnEdicion = null // Limpiar edición después de guardar
    },
    
    editarCita(cita) {
      this.citaEnEdicion = cita
      // Scroll suave al formulario
      this.$nextTick(() => {
        const formSection = document.querySelector('.form-section')
        if (formSection) {
          formSection.scrollIntoView({ behavior: 'smooth' })
        }
      })
    },
    
    cancelarEdicion() {
      this.citaEnEdicion = null
    }
  },
  mounted() {
    this.cargarCitas()
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
}

header {
  text-align: center;
  color: white;
  margin-bottom: 30px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  backdrop-filter: blur(10px);
}

header h1 {
  font-size: 2.5em;
  margin-bottom: 10px;
}

main {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 20px;
}

@media (max-width: 768px) {
  main {
    grid-template-columns: 1fr;
  }
  
  header h1 {
    font-size: 2em;
  }
}
</style>