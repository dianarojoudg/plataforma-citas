<template>
  <div class="table-container">
    <table>
      <thead>
        <tr>
          <th>Nombre Completo</th>
          <th>Teléfono</th>
          <th>Fecha</th>
          <th>Hora</th>
          <th>Motivo</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="cita in citas" :key="cita.id">
          <td>{{ cita.nombre }}</td>
          <td>{{ cita.telefono }}</td>
          <td>{{ cita.fecha }}</td>
          <td>{{ cita.hora }}</td>
          <td>{{ cita.motivo }}</td>
          <td>
            <button class="btn-delete" @click="eliminarCita(cita.id)">Eliminar</button>
          </td>
        </tr>
        <tr v-if="citas.length === 0">
          <td colspan="6" style="text-align: center;">No hay citas registradas</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'TablaPacientes',
  props: {
    citas: {
      type: Array,
      required: true
    }
  },
  methods: {
    eliminarCita(id) {
      this.$emit('eliminar', id)
    }
  }
}
</script>

<style scoped>
.table-container {
  overflow-x: auto;
  margin-bottom: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 12px;
  text-align: left;
  font-weight: 500;
}

td {
  padding: 12px;
  border-bottom: 1px solid #ddd;
}

tr:hover {
  background-color: #f5f5f5;
}

.btn-delete {
  padding: 5px 10px;
  background-color: #ff4757;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
}

.btn-delete:hover {
  background-color: #ff6b81;
}
</style>