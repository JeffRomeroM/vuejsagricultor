<template>
    <div class="app-container">
        <RouterLink to="/" class="link">Inicio</RouterLink>
      <div v-if="!autenticado" class="login-container">
        <h2>Iniciar sesión</h2>
        <input v-model="username" placeholder="Usuario" />
        <input v-model="password" type="password" placeholder="Contraseña" />
        <button @click="login">Ingresar</button>
        <p v-if="error" class="error">{{ error }}</p>
      </div>
  
      <div v-else class="crud-container">
        <h2>Gestión de Productos</h2>
        <button @click="nuevoProducto" class="agregarbtn">➕ Agregar Producto</button>
  
        <form v-if="mostrarFormulario" @submit.prevent="guardarProducto" class="formulario">
          <input v-model="productoEditar.nombre" placeholder="Nombre" required />
          <input v-model="productoEditar.ingrediente" placeholder="Ingrediente" required />
          <input v-model="productoEditar.precioRecurente" type="number" placeholder="Precio Rec." required />
          <input v-model="productoEditar.precioCasual" type="number" placeholder="Precio Cas." required />
          <input v-model="productoEditar.existencias" type="number" placeholder="Existencias" required />
          <button type="submit">{{ editIndex === null ? 'Agregar' : 'Actualizar' }}</button>
          <button type="button" @click="cancelarEdicion">Cancelar</button>
        </form>
  
        <input v-model="busqueda" placeholder="Buscar productos..." class="buscador" />
  
        <div class="tabla-responsive">
          <table class="tabla">
            <thead>
              <tr>
                <th>Nombre</th>
                <th>Ingrediente</th>
                <th>Precio Rec</th>
                <th>Precio Cas</th>
                <th>Existencias</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(p, i) in productosFiltrados" :key="i">
                <td>{{ p.nombre }}</td>
                <td>{{ p.ingrediente }}</td>
                <td>{{ p.precioRecurente }}</td>
                <td>{{ p.precioCasual }}</td>
                <td>{{ p.existencias }}</td>
                <td>
                  <button @click="cargarProducto(i)">✏️</button>
                  <button @click="eliminarProducto(i)">🗑️</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  const API_URL = 'https://script.google.com/macros/s/AKfycbyQmeDo0JmfbL5N-7bQ3WNgkTciCasEZtFpYL1Utp52NfnvhRByDfyaaNxJwSi-s8C-/exec';
  
  export default {
    data() {
      return {
        autenticado: false,
        username: '',
        password: '',
        error: '',
        productos: [],
        busqueda: '',
        mostrarFormulario: false,
        editIndex: null,
        productoEditar: {
          nombre: '',
          ingrediente: '',
          precioRecurente: '',
          precioCasual: '',
          existencias: '',
          fila: ''
        }
      };
    },
    computed: {
      productosFiltrados() {
        return this.productos.filter(p =>
          p.nombre.toLowerCase().includes(this.busqueda.toLowerCase()) ||
          p.ingrediente.toLowerCase().includes(this.busqueda.toLowerCase())
        );
      }
    },
    methods: {
      login() {
        if (this.username === 'admin' && this.password === '1234') {
          this.autenticado = true;
          this.cargarProductos();
        } else {
          this.error = 'Credenciales incorrectas';
        }
      },
      async cargarProductos() {
        try {
          const res = await fetch(API_URL);
          const data = await res.json();
          this.productos = data.slice(1).map((p, i) => ({
            nombre: p[0],
            ingrediente: p[1],
            precioRecurente: p[2],
            precioCasual: p[3],
            existencias: p[4],
            fila: i + 2
          }));
        } catch (err) {
          console.error("Error al cargar productos:", err);
        }
      },
      nuevoProducto() {
        this.editIndex = null;
        this.productoEditar = {
          nombre: '',
          ingrediente: '',
          precioRecurente: '',
          precioCasual: '',
          existencias: '',
          fila: ''
        };
        this.mostrarFormulario = true;
      },
      async guardarProducto() {
        const p = this.productoEditar;
        const params = new URLSearchParams({
          action: this.editIndex != null ? 'edit' : 'add',
          Nombre: p.nombre,
          IngredienteActivo: p.ingrediente,
          PrecioRecurente: p.precioRecurente,
          PrecioCasual: p.precioCasual,
          Existencias: p.existencias,
          row: p.fila || ''
        });
  
        await fetch(`${API_URL}?${params.toString()}`, { method: 'POST' });
        await this.cargarProductos();
        this.cancelarEdicion();
      },
      cargarProducto(index) {
        this.editIndex = index;
        this.productoEditar = { ...this.productos[index], editIndex: index };
        this.mostrarFormulario = true;
      },
      async eliminarProducto(index) {
        const fila = this.productos[index].fila;
        if (!confirm('¿Eliminar este producto?')) return;
  
        const params = new URLSearchParams({ action: 'delete', row: fila });
        await fetch(`${API_URL}?${params.toString()}`, { method: 'POST' });
        await this.cargarProductos();
      },
      cancelarEdicion() {
        this.mostrarFormulario = false;
        this.editIndex = null;
        this.productoEditar = {
          nombre: '',
          ingrediente: '',
          precioRecurente: '',
          precioCasual: '',
          existencias: '',
          fila: ''
        };
      }
    }
  };
  </script>
  
  <style scoped>
  .app-container {
    padding: 20px;
    font-family: Arial, sans-serif;
  }
  
  .login-container,
  .crud-container {
    max-width: 800px;
    margin: auto;
  }
  
  input {
    display: block;
    margin: 10px 0;
    padding: 8px;
    width: 100%;
    box-sizing: border-box;
  }
  
  button {
    padding: 10px 20px;
    margin: 5px 5px 0 0;
    cursor: pointer;
    background: #4CAF50;
    border: none;
    color: white;
    border-radius: 4px;
  }
  
  .error {
    color: red;
    text-align: center;
  }
  
  .tabla-responsive {
    overflow-x: auto;
    margin-top: 1rem;
  }
  
  .tabla {
    width: 100%;
    border-collapse: collapse;
    background: white;
  }
  
  th, td {
    border: 1px solid #ccc;
    padding: 10px;
    text-align: center;
  }
  
  th {
    background: #4CAF50;
    color: white;
  }
  
  tr:nth-child(even) {
    background-color: #f2f2f2;
  }
  
  .agregarbtn {
    background-color: green;
    border: none;
    width: 30%;
    margin: 10px auto;
    display: block;
    border-radius: 6px;
    color: white;
  }
  </style>
  