<template>
  <section class="form-section">
    <h2>📝 Registrar Nuevo Paciente</h2>
    <form @submit.prevent="agregarCita">
      <div class="form-group">
        <label for="nombre">Nombre Completo:</label>
        <input 
          type="text" 
          id="nombre" 
          v-model="nuevaCita.nombre" 
          required 
          placeholder="Ej: Juan Pérez González"
        >
      </div>

      <div class="form-group">
        <label for="telefono">Número Telefónico:</label>
        <input 
          type="tel" 
          id="telefono" 
          v-model="nuevaCita.telefono" 
          required 
          placeholder="Ej: 55-1234-5678"
        >
      </div>

      <div class="form-group">
        <label for="fecha">Fecha de la Cita:</label>
        <input 
          type="date" 
          id="fecha" 
          v-model="nuevaCita.fecha" 
          required
        >
      </div>

      <div class="form-group">
        <label for="hora">Hora de la Cita:</label>
        <input 
          type="time" 
          id="hora" 
          v-model="nuevaCita.hora" 
          required
        >
      </div>

      <div class="form-group">
        <label for="motivo">Motivo de la Consulta:</label>
        <textarea 
          id="motivo" 
          v-model="nuevaCita.motivo" 
          rows="3" 
          placeholder="Describa brevemente el motivo de la consulta"
        ></textarea>
      </div>

      <button type="submit" class="btn-primary">Registrar Cita</button>
    </form>
  </section>
</template>

<script>
export default {
  name: 'FormularioCita',
  data() {
    return {
      nuevaCita: {
        nombre: '',
        telefono: '',
        fecha: '',
        hora: '',
        motivo: ''
      }
    }
  },
  methods: {
    agregarCita() {
      const citas = JSON.parse(localStorage.getItem('citas')) || []
      
      const citaCompleta = {
        ...this.nuevaCita,
        id: Date.now()
      }
      
      citas.push(citaCompleta)
      localStorage.setItem('citas', JSON.stringify(citas))
      
      // Resetear formulario
      this.nuevaCita = {
        nombre: '',
        telefono: '',
        fecha: '',
        hora: '',
        motivo: ''
      }
      
      this.mostrarNotificacion('Cita registrada exitosamente', 'success')
      this.$emit('cita-agregada')
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

.btn-primary {
  width: 100%;
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