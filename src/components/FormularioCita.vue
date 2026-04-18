<template>
  <section class="form-section">
    <h2>📝 Registrar Nuevo Paciente</h2>

    <input v-model="nombre" placeholder="Nombre completo" />
    <input v-model="telefono" placeholder="Teléfono (10 dígitos)" />
    <input v-model="fecha" type="date" />
    <input v-model="hora" type="time" />
    <textarea v-model="motivo" placeholder="Motivo"></textarea>

    <button @click="guardarCita">Registrar Cita</button>
  </section>
</template>

<script>
export default {
  data() {
    return {
      nombre: '',
      telefono: '',
      fecha: '',
      hora: '',
      motivo: ''
    }
  },
  methods: {
    validarFormulario() {
      if (this.nombre.length < 5) {
        alert('Nombre mínimo 5 caracteres')
        return false
      }

      if (!/^\d{10}$/.test(this.telefono)) {
        alert('Teléfono inválido')
        return false
      }

      if (!this.fecha || !this.hora) {
        alert('Fecha y hora obligatorias')
        return false
      }

      return true
    },

    guardarCita() {
      if (!this.validarFormulario()) return

      let citas = JSON.parse(localStorage.getItem('citas')) || []

      citas.push({
        id: Date.now(),
        nombre: this.nombre,
        telefono: this.telefono,
        fecha: this.fecha,
        hora: this.hora,
        motivo: this.motivo
      })

      localStorage.setItem('citas', JSON.stringify(citas))

      this.$emit('cita-agregada')

      this.nombre = ''
      this.telefono = ''
      this.fecha = ''
      this.hora = ''
      this.motivo = ''
    }
  }
}
</script>

<style>
.form-section input,
.form-section textarea {
  width: 100%;
  margin-bottom: 10px;
  padding: 8px;
}
button {
  width: 100%;
  padding: 10px;
  background: purple;
  color: white;
  border: none;
}
</style>