<template>
  <section class="list-section">
    <h2>📋 Lista de Pacientes Registrados</h2>
    
    <!-- HU-07: BÚSQUEDA AVANZADA -->
    <div class="search-advanced">
      <div class="search-bar">
        <input 
          type="text" 
          v-model="terminoBusqueda" 
          placeholder="🔍 Buscar por nombre, teléfono, fecha, hora o motivo..."
          @input="realizarBusqueda"
          class="search-input"
        >
        <button v-if="terminoBusqueda" @click="limpiarBusqueda" class="btn-clear">✖</button>
      </div>
      
      <!-- Filtros rápidos -->
      <div class="filtros-rapidos">
        <button 
          v-for="filtro in filtrosRapidos" 
          :key="filtro.valor"
          :class="['filtro-btn', { active: filtroActivo === filtro.valor }]"
          @click="aplicarFiltroRapido(filtro.valor)"
        >
          {{ filtro.texto }}
        </button>
      </div>
      
      <!-- Tipo de búsqueda -->
      <div class="search-type">
        <label>
          <input type="radio" value="todos" v-model="tipoBusqueda"> Buscar en todos los campos
        </label>
        <label>
          <input type="radio" value="nombre" v-model="tipoBusqueda"> Solo nombre
        </label>
        <label>
          <input type="radio" value="telefono" v-model="tipoBusqueda"> Solo teléfono
        </label>
      </div>
    </div>
    
    <!-- Selector de Vista (Semanal/Mensual) -->
    <div class="view-selector">
      <button 
        :class="{ active: vista === 'semanal' }" 
        @click="vista = 'semanal'"
      >
        📅 Vista Semanal
      </button>
      <button 
        :class="{ active: vista === 'mensual' }" 
        @click="vista = 'mensual'"
      >
        🗓️ Vista Mensual
      </button>
      <button 
        :class="{ active: vista === 'tabla' }" 
        @click="vista = 'tabla'"
      >
        📊 Vista Tabla
      </button>
    </div>
    
    <!-- Resultados de búsqueda -->
    <div v-if="terminoBusqueda" class="resultados-busqueda">
      <span>🔍 {{ citasFiltradas.length }} resultado(s) encontrado(s) para "{{ terminoBusqueda }}"</span>
    </div>
    
    <!-- VISTA SEMANAL -->
    <div v-if="vista === 'semanal'" class="semanal-view">
      <div class="week-navigation">
        <button @click="cambiarSemana(-1)" class="btn-nav">◀ Semana anterior</button>
        <span class="week-range">{{ semanaActualTexto }}</span>
        <button @click="cambiarSemana(1)" class="btn-nav">Semana siguiente ▶</button>
      </div>
      <div class="week-grid">
        <div v-for="dia in diasSemana" :key="dia.fecha" class="day-card">
          <div class="day-header" :class="{ 'today': dia.esHoy }">
            {{ dia.nombre }}<br><small>{{ dia.fechaFormateada }}</small>
          </div>
          <div class="day-citas">
            <div v-for="cita in citasDelDia(dia.fecha)" :key="cita.id" class="cita-item" :class="{ 'highlight': resaltarCoincidencia(cita) }">
              <strong>{{ cita.hora }}</strong> - 
              <span v-html="resaltarTexto(cita.nombre)"></span>
              <div class="cita-actions">
                <button @click="abrirModalEdicion(cita)" class="btn-edit-sm">✏️</button>
                <button @click="eliminarCita(cita.id)" class="btn-delete-sm">🗑️</button>
              </div>
            </div>
            <div v-if="citasDelDia(dia.fecha).length === 0" class="no-citas">
              Sin citas
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- VISTA MENSUAL -->
    <div v-else-if="vista === 'mensual'" class="mensual-view">
      <div class="month-navigation">
        <button @click="cambiarMes(-1)" class="btn-nav">◀ Mes anterior</button>
        <span class="month-name">{{ mesActualNombre }}</span>
        <button @click="cambiarMes(1)" class="btn-nav">Mes siguiente ▶</button>
      </div>
      <div class="calendar-grid">
        <div class="calendar-header" v-for="dia in diasSemanaNombres" :key="dia">
          {{ dia }}
        </div>
        <div 
          v-for="dia in diasDelMes" 
          :key="dia.fecha" 
          class="calendar-day"
          :class="{ 
            'empty': !dia.fecha, 
            'today': dia.esHoy,
            'has-citas': dia.citas && dia.citas.length > 0
          }"
        >
          <div class="day-number">{{ dia.numero }}</div>
          <div v-if="dia.citas && dia.citas.length" class="day-citas-mini">
            <div v-for="cita in dia.citas.slice(0, 2)" :key="cita.id" class="cita-mini" :title="cita.nombre + ' - ' + cita.hora">
              {{ cita.hora }} {{ cita.nombre.slice(0, 15) }}
            </div>
            <div v-if="dia.citas.length > 2" class="cita-mas">
              +{{ dia.citas.length - 2 }} más
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- VISTA TABLA (mejorada con resaltado) -->
    <div v-else class="tabla-view">
      <div class="table-container">
        <table>
          <thead>
            <tr>
              <th @click="ordenarPor('fecha')">📅 Fecha 
                <span v-if="ordenCampo === 'fecha'">{{ ordenDireccion === 'asc' ? '↑' : '↓' }}</span>
              </th>
              <th @click="ordenarPor('nombre')">👤 Paciente
                <span v-if="ordenCampo === 'nombre'">{{ ordenDireccion === 'asc' ? '↑' : '↓' }}</span>
              </th>
              <th @click="ordenarPor('telefono')">📞 Teléfono</th>
              <th @click="ordenarPor('hora')">⏰ Hora
                <span v-if="ordenCampo === 'hora'">{{ ordenDireccion === 'asc' ? '↑' : '↓' }}</span>
              </th>
              <th>📝 Motivo</th>
              <th>⚡ Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="cita in citasOrdenadas" :key="cita.id" :class="{ 'highlight': resaltarCoincidencia(cita) }">
              <td>{{ formatFecha(cita.fecha) }}</td>
              <td v-html="resaltarTexto(cita.nombre)"></td>
              <td v-html="resaltarTexto(cita.telefono)"></td>
              <td v-html="resaltarTexto(cita.hora)"></td>
              <td v-html="resaltarTexto(cita.motivo)"></td>
              <td>
                <button class="btn-edit" @click="abrirModalEdicion(cita)">✏️ Editar</button>
                <button class="btn-delete" @click="eliminarCita(cita.id)">🗑️ Eliminar</button>
              </td>
            </tr>
            <tr v-if="citasOrdenadas.length === 0">
              <td colspan="6" style="text-align: center;">
                {{ terminoBusqueda ? 'No se encontraron resultados para "' + terminoBusqueda + '"' : 'No hay citas registradas' }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    
    <!-- Estadísticas mejoradas -->
    <div class="stats-container">
      <div class="stat-card">
        <h3>📊 Total Pacientes</h3>
        <p>{{ citas.length }}</p>
      </div>
      <div class="stat-card">
        <h3>📅 Citas Hoy</h3>
        <p>{{ citasHoy }}</p>
      </div>
      <div class="stat-card">
        <h3>🔍 Resultados</h3>
        <p>{{ citasFiltradas.length }}</p>
      </div>
    </div>
    
    <!-- Botón para modo oscuro/claro -->
    <div class="theme-toggle">
      <button @click="toggleTheme" class="btn-theme">
        {{ isDarkMode ? '☀️ Modo Claro' : '🌙 Modo Oscuro' }}
      </button>
    </div>
    
    <!-- Modal de Edición -->
    <div v-if="mostrarModal" class="modal" @click.self="cerrarModal">
      <div class="modal-content">
        <h3>✏️ Editar Cita</h3>
        <form @submit.prevent="guardarEdicion">
          <div class="form-group">
            <label>Nombre Completo:</label>
            <input v-model="citaEditando.nombre" required>
          </div>
          <div class="form-group">
            <label>Teléfono:</label>
            <input v-model="citaEditando.telefono" required>
          </div>
          <div class="form-group">
            <label>Fecha:</label>
            <input v-model="citaEditando.fecha" type="date" required>
          </div>
          <div class="form-group">
            <label>Hora:</label>
            <input v-model="citaEditando.hora" type="time" required>
          </div>
          <div class="form-group">
            <label>Motivo:</label>
            <textarea v-model="citaEditando.motivo" rows="2"></textarea>
          </div>
          <div class="modal-buttons">
            <button type="button" @click="cerrarModal" class="btn-cancel">Cancelar</button>
            <button type="submit" class="btn-confirm">Guardar Cambios</button>
          </div>
        </form>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: 'AgendaView',
  props: {
    citas: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      // Búsqueda
      terminoBusqueda: '',
      tipoBusqueda: 'todos',
      filtroActivo: 'todos',
      
      // Vista
      vista: 'tabla',
      fechaReferencia: new Date(),
      
      // Ordenamiento
      ordenCampo: 'fecha',
      ordenDireccion: 'asc',
      
      // Modal
      mostrarModal: false,
      citaEditando: null,
      
      // Tema
      isDarkMode: false,
      
      // Filtros rápidos
      filtrosRapidos: [
        { texto: '📋 Todos', valor: 'todos' },
        { texto: '📅 Hoy', valor: 'hoy' },
        { texto: '📆 Esta semana', valor: 'semana' },
        { texto: '📅 Este mes', valor: 'mes' }
      ]
    }
  },
  computed: {
    // Búsqueda filtrada
    citasFiltradas() {
      let resultado = this.citas
      
      // Aplicar filtro rápido primero
      if (this.filtroActivo !== 'todos') {
        const hoy = new Date().toISOString().split('T')[0]
        const hoyObj = new Date()
        
        if (this.filtroActivo === 'hoy') {
          resultado = resultado.filter(c => c.fecha === hoy)
        } else if (this.filtroActivo === 'semana') {
          const inicioSemana = this.getFechaInicioSemana(new Date())
          const finSemana = new Date(inicioSemana)
          finSemana.setDate(inicioSemana.getDate() + 6)
          const finStr = finSemana.toISOString().split('T')[0]
          resultado = resultado.filter(c => c.fecha >= inicioSemana.toISOString().split('T')[0] && c.fecha <= finStr)
        } else if (this.filtroActivo === 'mes') {
          const mes = hoyObj.getMonth()
          const año = hoyObj.getFullYear()
          resultado = resultado.filter(c => {
            const [cAño, cMes] = c.fecha.split('-')
            return parseInt(cAño) === año && parseInt(cMes) === mes + 1
          })
        }
      }
      
      // Aplicar búsqueda por texto
      if (!this.terminoBusqueda) return resultado
      
      const term = this.terminoBusqueda.toLowerCase()
      
      return resultado.filter(cita => {
        if (this.tipoBusqueda === 'nombre') {
          return cita.nombre.toLowerCase().includes(term)
        }
        if (this.tipoBusqueda === 'telefono') {
          return cita.telefono.includes(term)
        }
        // Búsqueda en todos los campos
        return cita.nombre.toLowerCase().includes(term) ||
               cita.telefono.includes(term) ||
               cita.fecha.includes(term) ||
               cita.hora.includes(term) ||
               (cita.motivo && cita.motivo.toLowerCase().includes(term))
      })
    },
    
    // Ordenamiento
    citasOrdenadas() {
      const orden = this.ordenDireccion === 'asc' ? 1 : -1
      return [...this.citasFiltradas].sort((a, b) => {
        let valA = a[this.ordenCampo]
        let valB = b[this.ordenCampo]
        
        if (this.ordenCampo === 'fecha') {
          valA = valA.split('-').join('')
          valB = valB.split('-').join('')
        }
        
        if (valA < valB) return -orden
        if (valA > valB) return orden
        return 0
      })
    },
    
    citasHoy() {
      const hoy = new Date().toISOString().split('T')[0]
      return this.citas.filter(cita => cita.fecha === hoy).length
    },
    
    // Vista semanal
    diasSemana() {
      const semana = []
      const inicioSemana = this.getFechaInicioSemana(this.fechaReferencia)
      const hoyStr = new Date().toISOString().split('T')[0]
      
      for (let i = 0; i < 7; i++) {
        const fecha = new Date(inicioSemana)
        fecha.setDate(inicioSemana.getDate() + i)
        const fechaStr = fecha.toISOString().split('T')[0]
        semana.push({
          nombre: this.getNombreDia(fecha.getDay()),
          fecha: fechaStr,
          fechaFormateada: `${fecha.getDate()}/${fecha.getMonth() + 1}`,
          esHoy: fechaStr === hoyStr
        })
      }
      return semana
    },
    
    semanaActualTexto() {
      const inicio = this.diasSemana[0].fechaFormateada
      const fin = this.diasSemana[6].fechaFormateada
      const [diaIni, mesIni] = inicio.split('/')
      const [diaFin, mesFin] = fin.split('/')
      if (mesIni === mesFin) {
        return `${diaIni} - ${diaFin} ${this.getNombreMes(parseInt(mesIni))}`
      }
      return `${diaIni} ${this.getNombreMes(parseInt(mesIni))} - ${diaFin} ${this.getNombreMes(parseInt(mesFin))}`
    },
    
    // Vista mensual
    diasSemanaNombres() {
      return ['Lun', 'Mar', 'Mié', 'Jue', 'Vie', 'Sáb', 'Dom']
    },
    
    diasDelMes() {
      const año = this.fechaReferencia.getFullYear()
      const mes = this.fechaReferencia.getMonth()
      const primerDia = new Date(año, mes, 1)
      const ultimoDia = new Date(año, mes + 1, 0)
      const diasEnMes = ultimoDia.getDate()
      const diaInicioSemana = primerDia.getDay() === 0 ? 6 : primerDia.getDay() - 1
      
      const hoyStr = new Date().toISOString().split('T')[0]
      const dias = []
      
      // Días vacíos al inicio
      for (let i = 0; i < diaInicioSemana; i++) {
        dias.push({ fecha: null, numero: '', esHoy: false, citas: [] })
      }
      
      // Días del mes
      for (let i = 1; i <= diasEnMes; i++) {
        const fechaStr = `${año}-${String(mes + 1).padStart(2, '0')}-${String(i).padStart(2, '0')}`
        const citasDelDia = this.citasFiltradas.filter(c => c.fecha === fechaStr)
        
        dias.push({
          fecha: fechaStr,
          numero: i,
          esHoy: fechaStr === hoyStr,
          citas: citasDelDia
        })
      }
      
      return dias
    },
    
    mesActualNombre() {
      return `${this.getNombreMes(this.fechaReferencia.getMonth() + 1)} ${this.fechaReferencia.getFullYear()}`
    }
  },
  mounted() {
    // Cargar preferencia de tema desde localStorage
    const temaGuardado = localStorage.getItem('darkMode')
    if (temaGuardado === 'true') {
      this.isDarkMode = true
      document.body.classList.add('dark-mode')
    }
  },
  methods: {
    // Búsqueda
    realizarBusqueda() {
      // El filtrado se hace automáticamente por el computed
    },
    
    limpiarBusqueda() {
      this.terminoBusqueda = ''
      this.tipoBusqueda = 'todos'
    },
    
    aplicarFiltroRapido(filtro) {
      this.filtroActivo = filtro
    },
    
    resaltarTexto(texto) {
      if (!this.terminoBusqueda || !texto) return texto
      const regex = new RegExp(`(${this.terminoBusqueda.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi')
      return texto.replace(regex, '<mark class="highlight-text">$1</mark>')
    },
    
    resaltarCoincidencia(cita) {
      if (!this.terminoBusqueda) return false
      const term = this.terminoBusqueda.toLowerCase()
      return cita.nombre.toLowerCase().includes(term) ||
             cita.telefono.includes(term) ||
             cita.fecha.includes(term) ||
             cita.hora.includes(term) ||
             (cita.motivo && cita.motivo.toLowerCase().includes(term))
    },
    
    // Ordenamiento
    ordenarPor(campo) {
      if (this.ordenCampo === campo) {
        this.ordenDireccion = this.ordenDireccion === 'asc' ? 'desc' : 'asc'
      } else {
        this.ordenCampo = campo
        this.ordenDireccion = 'asc'
      }
    },
    
    // Tema
    toggleTheme() {
      this.isDarkMode = !this.isDarkMode
      if (this.isDarkMode) {
        document.body.classList.add('dark-mode')
      } else {
        document.body.classList.remove('dark-mode')
      }
      localStorage.setItem('darkMode', this.isDarkMode)
    },
    
    // Utilidades
    formatFecha(fecha) {
      if (!fecha) return ''
      const [year, month, day] = fecha.split('-')
      return `${day}/${month}/${year}`
    },
    
    getNombreDia(dia) {
      const dias = ['Domingo', 'Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado']
      return dias[dia]
    },
    
    getNombreMes(mes) {
      const meses = ['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre']
      return meses[mes - 1]
    },
    
    getFechaInicioSemana(fecha) {
      const copia = new Date(fecha)
      const dia = copia.getDay()
      const diferencia = dia === 0 ? 6 : dia - 1
      copia.setDate(copia.getDate() - diferencia)
      copia.setHours(0, 0, 0, 0)
      return copia
    },
    
    cambiarSemana(delta) {
      const nuevaFecha = new Date(this.fechaReferencia)
      nuevaFecha.setDate(nuevaFecha.getDate() + delta * 7)
      this.fechaReferencia = nuevaFecha
    },
    
    cambiarMes(delta) {
      const nuevaFecha = new Date(this.fechaReferencia)
      nuevaFecha.setMonth(nuevaFecha.getMonth() + delta)
      this.fechaReferencia = nuevaFecha
    },
    
    citasDelDia(fecha) {
      return this.citasFiltradas
        .filter(cita => cita.fecha === fecha)
        .sort((a, b) => a.hora.localeCompare(b.hora))
    },
    
    // CRUD
    eliminarCita(id) {
      if(confirm('¿Estás seguro de eliminar esta cita?')) {
        const citas = JSON.parse(localStorage.getItem('citas')) || []
        const citasActualizadas = citas.filter(cita => cita.id !== id)
        localStorage.setItem('citas', JSON.stringify(citasActualizadas))
        this.mostrarNotificacion('✅ Cita eliminada correctamente', 'success')
        this.$emit('cita-eliminada')
      }
    },
    
    abrirModalEdicion(cita) {
      this.citaEditando = { ...cita }
      this.mostrarModal = true
    },
    
    cerrarModal() {
      this.mostrarModal = false
      this.citaEditando = null
    },
    
    guardarEdicion() {
      let citas = JSON.parse(localStorage.getItem('citas')) || []
      
      const existeDuplicado = citas.some(cita => 
        cita.id !== this.citaEditando.id &&
        cita.nombre.toLowerCase().trim() === this.citaEditando.nombre.toLowerCase().trim() &&
        cita.fecha === this.citaEditando.fecha &&
        cita.hora === this.citaEditando.hora
      )
      
      if (existeDuplicado) {
        this.mostrarNotificacion('⚠️ Ya existe otra cita con estos datos', 'error')
        return
      }
      
      const index = citas.findIndex(c => c.id === this.citaEditando.id)
      if (index !== -1) {
        citas[index] = this.citaEditando
        localStorage.setItem('citas', JSON.stringify(citas))
        this.mostrarNotificacion('✅ Cita actualizada correctamente', 'success')
        this.cerrarModal()
        this.$emit('cita-eliminada')
      }
    },
    
    mostrarNotificacion(mensaje, tipo) {
      const notification = document.createElement('div')
      notification.className = `notification ${tipo}`
      notification.textContent = mensaje
      document.body.appendChild(notification)
      setTimeout(() => notification.remove(), 3000)
    }
  }
}
</script>

<style scoped>
.list-section {
  background: white;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  transition: all 0.3s ease;
}

h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 1.5em;
  border-bottom: 2px solid #667eea;
  padding-bottom: 10px;
}

/* Búsqueda avanzada */
.search-advanced {
  margin-bottom: 20px;
}

.search-bar {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.search-input {
  flex: 1;
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 5px rgba(102, 126, 234, 0.3);
}

.btn-clear {
  margin-left: 10px;
  padding: 8px 12px;
  background: #ff4757;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.filtros-rapidos {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.filtro-btn {
  padding: 6px 12px;
  background: #f0f0f0;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.3s;
}

.filtro-btn.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.search-type {
  display: flex;
  gap: 15px;
  font-size: 12px;
  color: #666;
}

.search-type label {
  display: flex;
  align-items: center;
  gap: 4px;
  cursor: pointer;
}

.resultados-busqueda {
  margin-bottom: 15px;
  padding: 8px;
  background: #e3f2fd;
  border-radius: 5px;
  font-size: 13px;
}

/* Vista selector */
.view-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.view-selector button {
  flex: 1;
  padding: 10px;
  border: 1px solid #667eea;
  background: white;
  border-radius: 5px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s;
}

.view-selector button.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

/* Vista semanal */
.semanal-view {
  margin-bottom: 20px;
}

.week-navigation, .month-navigation {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 10px;
  background: #f5f5f5;
  border-radius: 8px;
}

.btn-nav {
  padding: 8px 15px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.week-range, .month-name {
  font-weight: bold;
  font-size: 1.1em;
}

.week-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 10px;
  overflow-x: auto;
}

.day-card {
  background: #f9f9f9;
  border-radius: 8px;
  padding: 10px;
  min-width: 120px;
}

.day-header {
  text-align: center;
  font-weight: bold;
  padding-bottom: 8px;
  border-bottom: 1px solid #ddd;
  margin-bottom: 8px;
}

.day-header.today {
  background: linear-gradient(135deg, #667eea20 0%, #764ba220 100%);
  border-radius: 5px;
  color: #667eea;
}

.day-citas {
  min-height: 100px;
}

.cita-item {
  background: white;
  padding: 5px;
  margin-bottom: 5px;
  border-radius: 4px;
  font-size: 12px;
  border-left: 3px solid #667eea;
}

.cita-item.highlight {
  background: #fff3cd;
  border-left-color: #ffc107;
}

.cita-actions {
  display: flex;
  gap: 5px;
  margin-top: 3px;
}

.btn-edit-sm, .btn-delete-sm {
  padding: 2px 6px;
  font-size: 10px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

.btn-edit-sm {
  background-color: #4CAF50;
  color: white;
}

.btn-delete-sm {
  background-color: #ff4757;
  color: white;
}

.no-citas {
  color: #999;
  font-size: 11px;
  text-align: center;
  padding: 10px 0;
}

/* Vista mensual (calendario) */
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
  background: white;
  border-radius: 8px;
  overflow: hidden;
}

.calendar-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 10px;
  text-align: center;
  font-weight: bold;
}

.calendar-day {
  min-height: 100px;
  border: 1px solid #eee;
  padding: 5px;
  background: white;
}

.calendar-day.empty {
  background: #f9f9f9;
}

.calendar-day.today {
  background: #e3f2fd;
  border: 2px solid #667eea;
}

.calendar-day.has-citas {
  background: #fef9e6;
}

.day-number {
  font-weight: bold;
  margin-bottom: 5px;
  font-size: 14px;
}

.day-citas-mini {
  font-size: 10px;
}

.cita-mini {
  background: #667eea20;
  padding: 2px 4px;
  margin-bottom: 2px;
  border-radius: 3px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.cita-mas {
  color: #667eea;
  font-size: 10px;
  margin-top: 2px;
}

/* Vista tabla */
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
  cursor: pointer;
  user-select: none;
}

th:hover {
  opacity: 0.9;
}

td {
  padding: 12px;
  border-bottom: 1px solid #ddd;
}

tr:hover {
  background-color: #f5f5f5;
}

tr.highlight {
  background-color: #fff3cd;
}

.btn-edit {
  padding: 5px 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  margin-right: 5px;
  font-size: 12px;
}

.btn-edit:hover {
  background-color: #45a049;
}

.btn-delete {
  padding: 5px 10px;
  background-color: #ff4757;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
}

.btn-delete:hover {
  background-color: #ff6b81;
}

/* Estadísticas */
.stats-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
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

/* Botón tema */
.theme-toggle {
  margin-top: 15px;
  text-align: center;
}

.btn-theme {
  padding: 8px 20px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 12px;
}

/* Resaltado de búsqueda */
:deep(.highlight-text) {
  background-color: #ffeb3b;
  color: #333;
  padding: 0 2px;
  border-radius: 3px;
}

/* Modal */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 25px;
  border-radius: 10px;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-buttons {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.btn-cancel {
  flex: 1;
  padding: 10px;
  background-color: #ddd;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-confirm {
  flex: 1;
  padding: 10px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.form-group input, .form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
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

.notification.error {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
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

/* Modo oscuro */
:global(body.dark-mode) {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
}

:global(body.dark-mode .list-section) {
  background: #2d2d3a;
  color: #fff;
}

:global(body.dark-mode .list-section h2) {
  color: #fff;
  border-bottom-color: #667eea;
}

:global(body.dark-mode .day-card) {
  background: #3a3a4a;
  color: #fff;
}

:global(body.dark-mode .cita-item) {
  background: #4a4a5a;
  color: #fff;
}

:global(body.dark-mode .stat-card) {
  background: linear-gradient(135deg, #3a3a4a 0%, #2d2d3a 100%);
}

:global(body.dark-mode .stat-card h3) {
  color: #ccc;
}

:global(body.dark-mode .stat-card p) {
  color: #fff;
}

:global(body.dark-mode table) {
  background: #2d2d3a;
  color: #fff;
}

:global(body.dark-mode td) {
  border-bottom-color: #4a4a5a;
}

:global(body.dark-mode tr:hover) {
  background-color: #3a3a4a;
}

:global(body.dark-mode .search-input) {
  background: #3a3a4a;
  color: #fff;
  border-color: #4a4a5a;
}

:global(body.dark-mode .filtro-btn) {
  background: #3a3a4a;
  color: #fff;
}

@media (max-width: 768px) {
  .week-grid {
    grid-template-columns: repeat(7, 100px);
  }
  
  .stats-container {
    grid-template-columns: 1fr;
  }
  
  .view-selector {
    flex-direction: column;
  }
}
</style>