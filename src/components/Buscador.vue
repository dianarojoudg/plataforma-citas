<template>
  <div class="search-bar">
    <input 
      type="text" 
      v-model="terminoBusqueda" 
      placeholder="Buscar por nombre o teléfono..."
      @input="buscar"
    >
  </div>
</template>

<script>
export default {
  name: 'Buscador',
  data() {
    return {
      terminoBusqueda: ''
    }
  },
  methods: {
    buscar() {
      this.$emit('buscar', this.terminoBusqueda)
    }
  }
}
</script>

<style scoped>
.search-bar {
  margin-bottom: 20px;
}

.search-bar input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
}

.search-bar input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 5px rgba(102, 126, 234, 0.3);
}
</style>