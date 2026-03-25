<template>
  <section class="form-section">
    <h2>{{ modoEdicion ? '✏️ Editar Cita' : '📝 Registrar Nueva Cita' }}</h2>
    <form @submit.prevent="guardarCita">
      <div class="form-group">
        <label for="nombre">Nombre Completo:</label>
        <input 
          type="text" 
          id="nombre" 
          v-model="citaForm.nombre" 
          required 
          placeholder="Ej: Juan Pérez González"
        >
      </div>

      <div class="form-group">
        <label for="telefono">Número Telefónico:</label>
        <input 
          type="tel" 
          id="telefono" 
          v-model="citaForm.telefono" 
          required 
          placeholder="Ej: 55-1234-5678"
        >
      </div>

      <div class="form-group">
        <label for="fecha">Fecha de la Cita:</label>
        <input 
          type="date" 
          id="fecha" 
          v-model="citaForm.fecha" 
          required
        >
      </div>

      <div class="form-group">
        <label for="hora">Hora de la Cita:</label>
        <input 
          type="time" 
          id="hora" 
          v-model="citaForm.hora" 
          required
        >
      </div>

      <div class="form-group">
        <label for="motivo">Motivo de la Consulta:</label>
        <textarea 
          id="motivo" 
          v-model="citaForm.motivo" 
          rows="3" 
          placeholder="Describa brevemente el motivo de la consulta"
        ></textarea>
      </div>

      <div class="form-buttons">
        <button type="submit" class="btn-primary">
          {{ modoEdicion ? 'Actualizar Cita' : 'Registrar Cita' }}
        </button>
        <button v-if="modoEdicion" type="button" class="btn-secondary" @click="cancelarEdicion">
          Cancelar
        </button>
      </div>
    </form>
  </section>
</template>

<script>
import Swal from 'sweetalert2'

export default {
  name: 'FormularioCita',
  props: {
    citaParaEditar: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      citaForm: {
        id: null,
        nombre: '',
        telefono: '',
        fecha: '',
        hora: '',
        motivo: ''
      }
    }
  },
  computed: {
    modoEdicion() {
      return this.citaParaEditar !== null
    }
  },
  watch: {
    citaParaEditar: {
      handler(nuevoValor) {
        if (nuevoValor) {
          this.citaForm = { ...nuevoValor }
        } else {
          this.resetearFormulario()
        }
      },
      immediate: true
    }
  },
  methods: {
    // Verificar si existe cita duplicada (excluyendo la cita actual en edición)
    existeCitaDuplicada(fecha, hora, idExcluir = null) {
      const citas = JSON.parse(localStorage.getItem('citas')) || []
      return citas.some(cita => 
        cita.fecha === fecha && 
        cita.hora === hora && 
        cita.id !== idExcluir
      )
    },

    guardarCita() {
      const citas = JSON.parse(localStorage.getItem('citas')) || []
      
      // Validación de campos obligatorios
      if (!this.citaForm.nombre || !this.citaForm.telefono || 
          !this.citaForm.fecha || !this.citaForm.hora) {
        this.mostrarNotificacion('Todos los campos son obligatorios', 'error')
        return
      }

      // Validación de duplicados
      if (this.existeCitaDuplicada(this.citaForm.fecha, this.citaForm.hora, this.citaForm.id)) {
        this.mostrarNotificacion('Ya existe una cita en esta fecha y hora', 'error')
        return
      }

      if (this.modoEdicion) {
        // Modo edición: actualizar cita existente
        const index = citas.findIndex(c => c.id === this.citaForm.id)
        if (index !== -1) {
          citas[index] = { ...this.citaForm }
          localStorage.setItem('citas', JSON.stringify(citas))
          this.mostrarNotificacion('Cita actualizada exitosamente', 'success')
          this.$emit('cita-actualizada')
          this.$emit('cancelar-edicion')
        }
      } else {
        // Modo creación: agregar nueva cita
        const nuevaCita = {
          ...this.citaForm,
          id: Date.now()
        }
        
        citas.push(nuevaCita)
        localStorage.setItem('citas', JSON.stringify(citas))
        this.mostrarNotificacion('Cita registrada exitosamente', 'success')
        this.$emit('cita-agregada')
        this.resetearFormulario()
      }
    },
    
    cancelarEdicion() {
      this.$emit('cancelar-edicion')
      this.resetearFormulario()
    },

    resetearFormulario() {
      this.citaForm = {
        id: null,
        nombre: '',
        telefono: '',
        fecha: '',
        hora: '',
        motivo: ''
      }
    },
    
    mostrarNotificacion(mensaje, tipo) {
      Swal.fire({
        title: 'Consultorio Médico',
        text: mensaje,
        icon: tipo === 'error' ? 'error' : 'success',
        confirmButtonText: 'Aceptar',
        confirmButtonColor: tipo === 'error' ? '#ff6b6b' : '#667eea'
      })
    }
  }
}
</script>

<style scoped>
.form-section {
  background: white;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 1.5em;
  border-bottom: 2px solid #667eea;
  padding-bottom: 10px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  color: #555;
  font-weight: 500;
}

input, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
  transition: border-color 0.3s;
}

input:focus, textarea:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 5px rgba(102, 126, 234, 0.3);
}

.form-buttons {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.btn-primary {
  flex: 1;
  padding: 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.btn-secondary {
  padding: 12px 20px;
  background: #6c757d;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.btn-secondary:hover {
  background: #5a6268;
}
</style>