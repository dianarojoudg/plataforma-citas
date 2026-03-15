<template>
  <div class="stats-container">
    <div class="stat-card">
      <h3>Total Pacientes</h3>
      <p>{{ totalPacientes }}</p>
    </div>
    <div class="stat-card">
      <h3>Citas Hoy</h3>
      <p>{{ citasHoy }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Estadisticas',
  props: {
    citas: {
      type: Array,
      required: true
    }
  },
  computed: {
    totalPacientes() {
      return this.citas.length
    },
    
    citasHoy() {
      const hoy = new Date().toISOString().split('T')[0]
      return this.citas.filter(cita => cita.fecha === hoy).length
    }
  }
}
</script>

<style scoped>
.stats-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-top: 20px;
}

.stat-card {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding: 15px;
  border-radius: 8px;
  text-align: center;
}

.stat-card h3 {
  color: #555;
  font-size: 14px;
  margin-bottom: 5px;
}

.stat-card p {
  color: #333;
  font-size: 24px;
  font-weight: bold;
}
</style>