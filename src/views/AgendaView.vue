<template>
  <section class="list-section">
    <h2>📋 Agenda de Pacientes</h2>

    <!-- 🔍 BUSQUEDA -->
    <div class="top-bar">
      <input 
        type="text" 
        v-model="terminoBusqueda"
        placeholder="🔍 Buscar paciente..."
      />

      <button class="btn-pdf" @click="exportarPDF">
        📄 PDF
      </button>
    </div>

    <!-- 📅 FILTROS -->
    <div class="filtros">
      <button :class="{active: filtro==='todos'}" @click="filtro='todos'">Todos</button>
      <button :class="{active: filtro==='hoy'}" @click="filtro='hoy'">Hoy</button>
      <button :class="{active: filtro==='semana'}" @click="filtro='semana'">Semana</button>
      <button :class="{active: filtro==='mes'}" @click="filtro='mes'">Mes</button>
    </div>

    <!-- 👀 VISTA -->
    <div class="view-selector">
      <button :class="{active: vista==='tabla'}" @click="vista='tabla'">Tabla</button>
      <button :class="{active: vista==='semanal'}" @click="vista='semanal'">Semanal</button>
      <button :class="{active: vista==='mensual'}" @click="vista='mensual'">Mensual</button>
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
          <td>{{ formatFecha(cita.fecha) }}</td>
          <td>{{ cita.nombre }}</td>
          <td>{{ cita.telefono }}</td>
          <td>{{ cita.hora }}</td>
          <td>{{ cita.motivo }}</td>
          <td>
            <button class="edit" @click="$emit('cita-editar', cita)">✏️</button>
            <button class="delete" @click="eliminarCita(cita.id)">🗑️</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- 📅 SEMANAL -->
    <div v-if="vista === 'semanal'" class="grid">
      <div v-for="(dia, i) in semana" :key="i" class="card">
        <h4>{{ dia.nombre }}</h4>
        <p v-for="c in dia.citas" :key="c.id">
          {{ c.hora }} - {{ c.nombre }}
        </p>
      </div>
    </div>

    <!-- 🗓️ MENSUAL -->
    <div v-if="vista === 'mensual'" class="grid">
      <div v-for="dia in mes" :key="dia.numero" class="card">
        <strong>{{ dia.numero }}</strong>
        <p v-for="c in dia.citas" :key="c.id">
          {{ c.hora }}
        </p>
      </div>
    </div>
  </section>
</template>

<script>
import jsPDF from "jspdf"

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
      let res = this.citas

      const hoy = new Date()

      // FILTROS
      if (this.filtro === 'hoy') {
        const h = hoy.toISOString().split('T')[0]
        res = res.filter(c => c.fecha === h)
      }

      if (this.filtro === 'semana') {
        const inicio = new Date(hoy)
        inicio.setDate(hoy.getDate() - hoy.getDay())

        const fin = new Date(inicio)
        fin.setDate(inicio.getDate() + 6)

        res = res.filter(c => {
          const f = new Date(c.fecha)
          return f >= inicio && f <= fin
        })
      }

      if (this.filtro === 'mes') {
        res = res.filter(c => {
          const f = new Date(c.fecha)
          return f.getMonth() === hoy.getMonth()
        })
      }

      // BUSQUEDA
      if (this.terminoBusqueda) {
        res = res.filter(c =>
          c.nombre.toLowerCase().includes(this.terminoBusqueda.toLowerCase())
        )
      }

      return res
    },

    semana() {
      const dias = ['Dom','Lun','Mar','Mié','Jue','Vie','Sáb']
      return dias.map((d, i) => ({
        nombre: d,
        citas: this.citasFiltradas.filter(c => new Date(c.fecha).getDay() === i)
      }))
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

      doc.setFontSize(14)
      doc.text("Agenda de Pacientes", 10, 10)

      let y = 20
      this.citasFiltradas.forEach(c => {
        doc.text(`${c.fecha} | ${c.hora} | ${c.nombre}`, 10, y)
        y += 8
      })

      doc.save("agenda.pdf")
    },

    formatFecha(f) {
      const [y,m,d] = f.split('-')
      return `${d}/${m}/${y}`
    }
  }
}
</script>

<style scoped>
.list-section {
  background: white;
  padding: 25px;
  border-radius: 16px;
  box-shadow: 0 15px 40px rgba(0,0,0,0.08);
}

/* TOP */
.top-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

input {
  flex: 1;
  padding: 10px;
  border-radius: 10px;
  border: 1px solid #ddd;
}

.btn-pdf {
  background: #ff4b5c;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 10px;
  cursor: pointer;
}

/* FILTROS */
.filtros {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.filtros button {
  padding: 6px 12px;
  border-radius: 20px;
  border: none;
  cursor: pointer;
  background: #eee;
}

.filtros .active {
  background: #667eea;
  color: white;
}

/* VISTAS */
.view-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.view-selector button {
  flex: 1;
  padding: 10px;
  border-radius: 10px;
  border: 1px solid #ddd;
  cursor: pointer;
}

.view-selector .active {
  background: #667eea;
  color: white;
}

/* TABLA */
table {
  width: 100%;
  border-collapse: collapse;
}

th {
  background: #667eea;
  color: white;
  padding: 10px;
}

td {
  padding: 10px;
  border-bottom: 1px solid #eee;
}

tr:hover {
  background: #f9f9ff;
}

/* BOTONES */
.edit {
  background: #4CAF50;
  color: white;
  border: none;
  padding: 5px;
  margin-right: 5px;
}

.delete {
  background: #ff4b5c;
  color: white;
  border: none;
  padding: 5px;
}

/* GRID */
.grid {
  display: grid;
  grid-template-columns: repeat(7,1fr);
  gap: 10px;
}

.card {
  background: #f5f5ff;
  padding: 10px;
  border-radius: 10px;
}
</style>