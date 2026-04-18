<template>
  <div class="container">

    <header class="header">
      <h1>🏥 Consultorio Médico</h1>
      <p>Sistema de Registro y Visualización de Pacientes</p>
    </header>

    <main class="main-container">

      <div class="form-container">
        <FormularioCita 
          :citaParaEditar="citaEnEdicion"
          @cita-agregada="cargarCitas"
          @cita-actualizada="cargarCitas"
          @cancelar-edicion="cancelarEdicion"
        />
      </div>

      <div class="list-container">
        <AgendaView 
          :citas="citas" 
          @cita-eliminada="cargarCitas"
        />
      </div>

    </main>
  </div>
</template>

<script>
import FormularioCita from './components/FormularioCita.vue'
import AgendaView from './views/AgendaView.vue'

export default {
  components: { FormularioCita, AgendaView },
  data() {
    return {
      citas: [],
      citaEnEdicion: null
    }
  },
  methods: {
    cargarCitas() {
      this.citas = JSON.parse(localStorage.getItem('citas')) || []
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
body {
  margin: 0;
  font-family: 'Segoe UI';
  background: linear-gradient(135deg, #667eea, #764ba2);
}

.container {
  max-width: 1300px;
  margin: auto;
  padding: 20px;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 20px;
  padding: 20px;
  background: rgba(255,255,255,0.1);
  border-radius: 10px;
}

.main-container {
  display: flex;
  gap: 20px;
}

.form-container {
  width: 30%;
  background: white;
  padding: 20px;
  border-radius: 10px;
}

.list-container {
  width: 70%;
}
</style>