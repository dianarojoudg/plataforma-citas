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
/* Estilos globales para modo oscuro */
body {
  transition: background 0.3s ease;
}

body.dark-mode {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
}

body.dark-mode .form-section {
  background: #2d2d3a;
  color: #fff;
}

body.dark-mode .form-section h2 {
  color: #fff;
}

body.dark-mode .form-group label {
  color: #ccc;
}

body.dark-mode input, 
body.dark-mode textarea {
  background: #3a3a4a;
  color: #fff;
  border-color: #4a4a5a;
}

body.dark-mode input:focus, 
body.dark-mode textarea:focus {
  border-color: #667eea;
}

body.dark-mode .btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
</style>