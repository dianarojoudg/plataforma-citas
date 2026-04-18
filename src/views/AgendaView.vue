<template>
  <section class="list-section">
    <h2>📋 Agenda de Pacientes</h2>

    <!-- 🔍 BUSCADOR -->
    <input 
      type="text" 
      v-model="terminoBusqueda" 
      placeholder="Buscar paciente..."
      class="search-input"
    />

    <!-- 📅 FILTROS -->
    <div class="filtros">
      <button @click="filtro = 'todos'">Todos</button>
      <button @click="filtro = 'hoy'">Hoy</button>
      <button @click="filtro = 'semana'">Semana</button>
      <button @click="filtro = 'mes'">Mes</button>
    </div>

    <!-- 📄 BOTÓN PDF -->
    <button class="btn-pdf" @click="exportarPDF">📄 Descargar PDF</button>

    <!-- 👀 VISTA -->
    <div class="view-selector">
      <button @click="vista = 'tabla'">Tabla</button>
      <button @click="vista = 'semanal'">Semanal</button>
      <button @click="vista = 'mensual'">Mensual</button>
    </div>

    <!-- 📊 TABLA -->
    <table v-if="vista === 'tabla'">
      <thead>
        <tr>
          <th>Fecha</th>
          <th>Nombre</th>
          <th>Teléfono</th>
          <th>Hora</th>
          <th>Motivo</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="cita in citasFiltradas" :key="cita.id">
          <td>{{ cita.fecha }}</td>
          <td>{{ cita.nombre }}</td>
          <td>{{ cita.telefono }}</td>
          <td>{{ cita.hora }}</td>
          <td>{{ cita.motivo }}</td>
          <td>
            <button @click="$emit('cita-editar', cita)">✏️</button>
            <button @click="eliminarCita(cita.id)">🗑️</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- 📅 SEMANAL -->
    <div v-if="vista === 'semanal'" class="grid">
      <div v-for="dia in semana" :key="dia.fecha" class="card">
        <h4>{{ dia.nombre }}</h4>
        <p v-for="c in dia.citas" :key="c.id">
          {{ c.hora }} - {{ c.nombre }}
        </p>
      </div>
    </div>

    <!-- 🗓️ MENSUAL -->
    <div v-if="vista === 'mensual'" class="grid">
      <div v-for="dia in mes" :key="dia.fecha" class="card">
        <strong>{{ dia.numero }}</strong>
        <p v-for="c in dia.citas" :key="c.id">
          {{ c.hora }}
        </p>
      </div>
    </div>

  </section>
</template>

<script>
import jsPDF from 'jspdf'

export default {
  props: ['citas'],
  data() {
    return {
      terminoBusqueda: '',
      filtro: 'todos',
      vista: 'tabla'
    }
  },

  computed: {
    citasFiltradas() {
      let resultado = this.citas

      const hoy = new Date().toISOString().split('T')[0]

      // 🔎 FILTRO POR TIEMPO
      if (this.filtro === 'hoy') {
        resultado = resultado.filter(c => c.fecha === hoy)
      }

      if (this.filtro === 'semana') {
        const inicio = new Date()
        inicio.setDate(inicio.getDate() - inicio.getDay())
        const fin = new Date(inicio)
        fin.setDate(fin.getDate() + 6)

        resultado = resultado.filter(c => {
          const f = new Date(c.fecha)
          return f >= inicio && f <= fin
        })
      }

      if (this.filtro === 'mes') {
        const hoyObj = new Date()
        resultado = resultado.filter(c => {
          const f = new Date(c.fecha)
          return f.getMonth() === hoyObj.getMonth()
        })
      }

      // 🔍 BUSQUEDA
      if (this.terminoBusqueda) {
        resultado = resultado.filter(c =>
          c.nombre.toLowerCase().includes(this.terminoBusqueda.toLowerCase())
        )
      }

      return resultado
    },

    semana() {
      const dias = ['Dom', 'Lun', 'Mar', 'Mié', 'Jue', 'Vie', 'Sáb']
      return dias.map((d, i) => {
        return {
          nombre: d,
          citas: this.citasFiltradas.filter(c => new Date(c.fecha).getDay() === i)
        }
      })
    },

    mes() {
      const dias = []
      for (let i = 1; i <= 31; i++) {
        dias.push({
          numero: i,
          citas: this.citasFiltradas.filter(c => new Date(c.fecha).getDate() === i)
        })
      }
      return dias
    }
  },

  methods: {
    eliminarCita(id) {
      const nuevas = this.citas.filter(c => c.id !== id)
      localStorage.setItem('citas', JSON.stringify(nuevas))
      this.$emit('cita-eliminada')
    },

    exportarPDF() {
      const doc = new jsPDF()

      doc.text("Agenda de Pacientes", 10, 10)

      let y = 20
      this.citasFiltradas.forEach(c => {
        doc.text(`${c.fecha} - ${c.hora} - ${c.nombre}`, 10, y)
        y += 10
      })

      doc.save("agenda.pdf")
    }
  }
}
</script>

<style scoped>
.list-section {
  background: white;
  padding: 20px;
  border-radius: 10px;
}

.search-input {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
}

.filtros {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

button {
  padding: 6px 10px;
  cursor: pointer;
}

.btn-pdf {
  background: red;
  color: white;
  margin-bottom: 10px;
}

.view-selector {
  margin-bottom: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

td, th {
  border: 1px solid #ddd;
  padding: 8px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 10px;
}

.card {
  background: #f5f5f5;
  padding: 10px;
}
</style>