<script setup>
import { ref, watch } from 'vue'


const serviciosGuardados = localStorage.getItem('servicios-barberia')
const servicios = ref(serviciosGuardados ? JSON.parse(serviciosGuardados) : [])


watch(servicios, (nuevaLista) => {
  localStorage.setItem('servicios-barberia', JSON.stringify(nuevaLista))
}, { deep: true })


const preciosServicios = {
  'Corte clásico': 20000,
  'Corte moderno': 25000,
  'Barba': 15000,
  'Corte + barba': 35000,
  'Cejas': 8000,
  'Tinte': 40000
}


const mostrarModal = ref(false)
const modoEdicion = ref(false)
const mostrarConfirmacion = ref(false)

const idEliminar = ref(null)
const idEditar = ref(null)
const error = ref('')



const cliente = ref('')
const serviciosSeleccionados = ref([])
const barbero = ref('')
const fecha = ref('')
const hora = ref('')
const precio = ref('')
const metodoPago = ref('')
const estadoPago = ref('')
const observaciones = ref('')



watch(serviciosSeleccionados, (nuevosServicios) => {
  const sumaTotal = nuevosServicios.reduce((total, servicio) => {
    return total + (preciosServicios[servicio] || 0)
  }, 0)
  precio.value = sumaTotal
}, { deep: true })



function abrirModal() {
  limpiarFormulario()
  modoEdicion.value = false
  mostrarModal.value = true
}


function cerrarModal() {
  mostrarModal.value = false
  limpiarFormulario()
}


function limpiarFormulario() {
  cliente.value = ''
  serviciosSeleccionados.value = []
  barbero.value = ''
  fecha.value = ''
  hora.value = ''
  precio.value = ''
  metodoPago.value = ''
  estadoPago.value = ''
  observaciones.value = ''
  error.value = ''
  idEditar.value = null
}


function guardarServicio() {
  error.value = ''

  if (
    !cliente.value.trim() ||
    serviciosSeleccionados.value.length === 0 ||
    !barbero.value ||
    !fecha.value ||
    !hora.value ||
    precio.value === '' ||
    !metodoPago.value ||
    !estadoPago.value
  ) {
    error.value = 'Por favor complete todos los campos obligatorios y seleccione al menos un corte.'
    return
  }

  if (Number(precio.value) <= 0) {
    error.value = 'El precio debe ser mayor a 0.'
    return
  }

  const tipoServicioTexto = serviciosSeleccionados.value.join(', ')

  if (!modoEdicion.value) {
    servicios.value.push({
      id: Date.now(),
      cliente: cliente.value.trim(),
      tipoServicio: tipoServicioTexto,
      calificacion: 0,
      barbero: barbero.value,
      fecha: fecha.value,
      hora: hora.value,
      precio: Number(precio.value),
      metodoPago: metodoPago.value,
      estadoPago: estadoPago.value,
      observaciones: observaciones.value.trim()
    })
  } else {
    const index = servicios.value.findIndex(s => s.id === idEditar.value)
    if (index !== -1) {
      servicios.value[index] = {
        ...servicios.value[index],
        cliente: cliente.value.trim(),
        tipoServicio: tipoServicioTexto,
        barbero: barbero.value,
        fecha: fecha.value,
        hora: hora.value,
        precio: Number(precio.value),
        metodoPago: metodoPago.value,
        estadoPago: estadoPago.value,
        observaciones: observaciones.value.trim()
      }
    }
  }

  cerrarModal()
}



function calificarServicioPost(idServicio, nota) {
  const index = servicios.value.findIndex(s => s.id === idServicio)
  if (index !== -1) {
    servicios.value[index].calificacion = nota
  }
}

// Editar servicio
function editarServicio(servicio) {
  modoEdicion.value = true
  idEditar.value = servicio.id

  cliente.value = servicio.cliente
  serviciosSeleccionados.value = servicio.tipoServicio ? servicio.tipoServicio.split(', ') : []
  barbero.value = servicio.barbero
  fecha.value = servicio.fecha || ''
  hora.value = servicio.hora || ''
  precio.value = servicio.precio
  metodoPago.value = servicio.metodoPago
  estadoPago.value = servicio.estadoPago
  observaciones.value = servicio.observaciones || ''

  mostrarModal.value = true
}


function preguntarEliminar(id) {
  idEliminar.value = id
  mostrarConfirmacion.value = true
}

function eliminarServicio() {
  servicios.value = servicios.value.filter(s => s.id !== idEliminar.value)
  mostrarConfirmacion.value = false
  idEliminar.value = null
}

function cancelarEliminar() {
  mostrarConfirmacion.value = false
  idEliminar.value = null
}


function totalServicios() {
  return servicios.value.length
}

function totalVentas() {
  return servicios.value.reduce((acc, s) => acc + Number(s.precio || 0), 0)
}

function totalPendiente() {
  return servicios.value
    .filter(s => s.estadoPago === 'Pendiente' || s.estadoPago === 'Fiado')
    .reduce((acc, s) => acc + Number(s.precio || 0), 0)
}


function iconoPago(metodo) {
  if (metodo === 'Efectivo') return '💵'
  if (metodo === 'Transferencia') return '📱'
  if (metodo === 'Tarjeta') return '💳'
  return '💰'
}

function estrellas(numero) {
  return '★'.repeat(Math.max(0, Math.min(5, Number(numero) || 0))) + '☆'.repeat(5 - Math.max(0, Math.min(5, Number(numero) || 0)))
}
</script>

<template>

  <div class="app">

    <header>
      <div>
        <h1>✂️ Barbería Don Ramiro</h1>
        <p>Registro de servicios</p>
      </div>

      <button @click="abrirModal">
        + Registrar servicio
      </button>
    </header>



    <section class="estadisticas">
      <div class="estadistica">
        <span>Servicios</span>
        <h2>{{ totalServicios() }}</h2>
      </div>

      <div class="estadistica">
        <span>Ventas totales</span>
        <h2>${{ totalVentas() }}</h2>
      </div>

      <div class="estadistica pendiente">
        <span>Dinero pendiente</span>
        <h2>${{ totalPendiente() }}</h2>
      </div>
    </section>



    <section class="contenedor">
      <h2>Servicios registrados</h2>

      <div v-if="servicios.length === 0" class="sin-servicios">
        <h3>✂️ Aún no hay servicios</h3>
        <p>Registra el primer servicio de la barbería.</p>
      </div>

      <div v-else class="lista-servicios">
        <div
          v-for="servicio in servicios"
          :key="servicio.id"
          class="tarjeta"
          :class="{
            tarjetaPendiente: servicio.estadoPago === 'Pendiente',
            tarjetaFiado: servicio.estadoPago === 'Fiado',
            tarjetaBaja: servicio.calificacion > 0 && servicio.calificacion <= 2
          }"
        >
          <div class="tarjeta-header">
            <div>
              <h3>{{ servicio.cliente }}</h3>
              <p class="corte-detalle">
                <b>✂️ Corte / Servicio:</b> {{ servicio.tipoServicio }}
              </p>
            </div>

            <span v-if="servicio.estadoPago === 'Pagado'" class="pagado">
              Pagado
            </span>
            <span v-else-if="servicio.estadoPago === 'Pendiente'" class="pendiente">
              ⏳ Pendiente
            </span>
            <span v-else class="fiado">
              Fiado
            </span>
          </div>



          <div class="calificacion-caja">
            <span class="titulo-calificacion">Calificación del corte:</span>

            <!-- Si ya fue calificado: Muestra las estrellas guardadas -->
            <div v-if="servicio.calificacion > 0" class="casilla-estrellas" :class="{ 'baja-calificacion': servicio.calificacion <= 2 }">
              <span class="estrellas-iconos">{{ estrellas(servicio.calificacion) }}</span>
              <span class="nota-numero">({{ servicio.calificacion }}/5)</span>
            </div>



            <div v-else class="estrellas-selector-post">
              <span
                v-for="estrella in 5"
                :key="estrella"
                class="estrella-opcion-post"
                @click="calificarServicioPost(servicio.id, estrella)"
              >
                ★
              </span>
              <span class="indicacion-click">(Haz clic en una estrella)</span>
            </div>
          </div>

          <div class="informacion">
            <p>👨‍💼 <b>Barbero:</b> {{ servicio.barbero }}</p>
            <p>📅 <b>Fecha:</b> {{ servicio.fecha }} | ⏰ <b>Hora:</b> {{ servicio.hora }}</p>
            <p>💰 <b>Precio total:</b> ${{ servicio.precio }}</p>
            <p>{{ iconoPago(servicio.metodoPago) }} {{ servicio.metodoPago }}</p>

            <p v-if="servicio.observaciones">
              📝 {{ servicio.observaciones }}
            </p>
          </div>

          <div class="botones">
            <button class="editar" @click="editarServicio(servicio)">
              ✏️ Editar
            </button>
            <button class="eliminar" @click="preguntarEliminar(servicio.id)">
              🗑️ Eliminar
            </button>
          </div>
        </div>
      </div>
    </section>


    <div v-show="mostrarModal" class="modal-fondo">
      <div class="modal">

        <div class="modal-header">
          <h2>{{ modoEdicion ? 'Editar servicio' : 'Registrar servicio' }}</h2>
          <button class="cerrar" @click="cerrarModal">×</button>
        </div>

        <form @submit.prevent="guardarServicio">
          <p v-if="error" class="error">{{ error }}</p>

          <label>Nombre del cliente</label>
          <input type="text" v-model="cliente" placeholder="Ej: Juan Pérez">

          <label>Corte / Servicios realizados</label>
          <div class="checkbox-group">
            <label
              v-for="(valorPrecio, nombreServicio) in preciosServicios"
              :key="nombreServicio"
              class="checkbox-item"
            >
              <input
                type="checkbox"
                :value="nombreServicio"
                v-model="serviciosSeleccionados"
              >
              <span>{{ nombreServicio }} (${{ valorPrecio }})</span>
            </label>
          </div>

          <label>Barbero</label>
          <select v-model="barbero">
            <option value="">Seleccione</option>
            <option>Don Ramiro</option>
            <option>Empleado 1</option>
            <option>Empleado 2</option>
          </select>

          <label>Fecha</label>
          <input type="date" v-model="fecha">

          <label>Hora</label>
          <input type="time" v-model="hora">

          <label>Precio total</label>
          <input type="number" v-model="precio" placeholder="Se calcula automáticamente">

          <label>Método de pago</label>
          <select v-model="metodoPago">
            <option value="">Seleccione</option>
            <option>Efectivo</option>
            <option>Transferencia</option>
            <option>Tarjeta</option>
          </select>

          <label>Estado del pago</label>
          <select v-model="estadoPago">
            <option value="">Seleccione</option>
            <option>Pagado</option>
            <option>Pendiente</option>
            <option>Fiado</option>
          </select>

          <label>Observaciones</label>
          <textarea v-model="observaciones" placeholder="Observaciones opcionales..."></textarea>

          <div class="acciones-formulario">
            <button type="button" class="cancelar" @click="cerrarModal">Cancelar</button>
            <button type="submit">Guardar</button>
          </div>
        </form>

      </div>
    </div>

    
    <div v-show="mostrarConfirmacion" class="modal-fondo">
      <div class="confirmacion">
        <h2>¿Eliminar servicio?</h2>
        <p>Esta acción eliminará el registro permanentemente.</p>
        <div>
          <button @click="cancelarEliminar">Cancelar</button>
          <button class="eliminar" @click="eliminarServicio">Sí, eliminar</button>
        </div>
      </div>
    </div>

  </div>

</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background: #f4f4f4;
  font-family: Arial, sans-serif;
}

.app {
  min-height: 100vh;
}


header {
  background: #111;
  color: white;
  padding: 25px 8%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

header h1 {
  font-size: 28px;
}

header p {
  color: #bbb;
  margin-top: 5px;
}

/* BOTONES */
button {
  border: none;
  padding: 10px 16px;
  border-radius: 7px;
  cursor: pointer;
  background: #d4a017;
  color: white;
  font-weight: bold;
}

button:hover {
  opacity: 0.9;
}


.estadisticas {
  padding: 25px 8%;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.estadistica {
  background: white;
  padding: 20px;
  border-radius: 10px;
  border-left: 5px solid #d4a017;
}

.estadistica h2 {
  margin-top: 10px;
}

.estadistica.pendiente {
  border-left-color: #e74c3c;
}


.contenedor {
  padding: 10px 8% 40px;
}

.contenedor h2 {
  margin-bottom: 20px;
}


.lista-servicios {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
}

.tarjeta {
  background: white;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 3px 10px rgba(0,0,0,0.1);
  border-left: 5px solid #333;
}

.tarjetaPendiente {
  border-left-color: orange;
}

.tarjetaFiado {
  border-left-color: red;
}

.tarjetaBaja {
  background: #fff5f5;
}

.tarjeta-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.tarjeta-header h3 {
  font-size: 20px;
}

.corte-detalle {
  color: #555;
  margin-top: 5px;
  font-size: 15px;
}


.calificacion-caja {
  margin: 12px 0;
}

.titulo-calificacion {
  display: block;
  font-weight: bold;
  font-size: 14px;
  margin-bottom: 6px;
  color: #222;
}

.estrellas-selector-post {
  display: flex;
  align-items: center;
  gap: 6px;
  background: #fcfcfc;
  border: 1px solid #d1d1d1;
  border-radius: 8px;
  padding: 8px 12px;
}

.estrella-opcion-post {
  font-size: 24px;
  color: #ccc;
  cursor: pointer;
  transition: color 0.15s, transform 0.1s;
  user-select: none;
}

.estrella-opcion-post:hover {
  color: #f39c12;
  transform: scale(1.2);
}

.indicacion-click {
  font-size: 13px;
  color: #777;
  margin-left: 6px;
}

.casilla-estrellas {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: #fff8e1;
  border: 1px solid #ffe082;
  padding: 6px 12px;
  border-radius: 6px;
}

.casilla-estrellas.baja-calificacion {
  background: #ffebee;
  border-color: #ffcdd2;
}

.estrellas-iconos {
  color: #f39c12;
  font-size: 16px;
  letter-spacing: 2px;
}

.nota-numero {
  font-weight: bold;
  font-size: 13px;
  color: #444;
}

.informacion p {
  margin: 8px 0;
}



.pagado {
  background: #d4edda;
  color: #155724;
  padding: 5px 8px;
  border-radius: 5px;
  font-size: 14px;
  height: fit-content;
}

.pendiente {
  background: #fff3cd;
  color: #856404;
  padding: 5px 8px;
  border-radius: 5px;
  font-size: 14px;
  height: fit-content;
}

.fiado {
  background: #f8d7da;
  color: #721c24;
  padding: 5px 8px;
  border-radius: 5px;
  font-size: 14px;
  height: fit-content;
}


.botones {
  display: flex;
  gap: 10px;
  margin-top: 15px;
}

.editar {
  background: #3498db;
}

.eliminar {
  background: #e74c3c;
}


.modal-fondo {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  z-index: 100;
}

.modal,
.confirmacion {
  background: white;
  width: 100%;
  max-width: 550px;
  border-radius: 10px;
  padding: 25px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.cerrar {
  background: transparent;
  color: #333;
  font-size: 30px;
  padding: 0;
  line-height: 1;
}

form label {
  display: block;
  margin-top: 12px;
  margin-bottom: 5px;
  font-weight: bold;
}

/* SELECCIÓN MÚLTIPLE (CHECKBOXES) */
.checkbox-group {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 10px;
  background: #f9f9f9;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

.checkbox-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: normal;
  margin: 0;
  cursor: pointer;
}

.checkbox-item input {
  width: auto;
  cursor: pointer;
}

input,
select,
textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

textarea {
  min-height: 80px;
  resize: vertical;
}

.error {
  background: #f8d7da;
  color: #721c24;
  padding: 10px;
  border-radius: 5px;
  margin-bottom: 10px;
}

.acciones-formulario {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.cancelar {
  background: #777;
}

.confirmacion {
  text-align: center;
}

.confirmacion p {
  margin: 15px 0;
}

.confirmacion div {
  display: flex;
  justify-content: center;
  gap: 10px;
}


.sin-servicios {
  text-align: center;
  background: white;
  padding: 50px;
  border-radius: 10px;
}



@media (max-width: 600px) {
  header {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }

  .estadisticas {
    grid-template-columns: 1fr;
  }
}
</style>