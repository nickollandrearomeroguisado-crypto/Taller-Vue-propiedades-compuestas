<script setup>
import { ref, reactive, computed } from 'vue'

const Cupo_Maximo = 10
const carreras = ['Sistemas', 'Diseño', 'Administración', 'Contabilidad']
const estudiantes = ref([])
let siguienteId= 1 //contador del id 

//Formulario
const formulario = reactive({
    nombre: '', 
    carrera: '',
    calificacion: ''
})

const errorNombre = computed(() => {
    if(formulario.nombre === '') return "El nombre es obligatorio"
    if(formulario.nombre.length < 3) return "El nombre debe tener al menos 3 letras"
    return ''
})

const errorCarrera = computed(() => 
    formulario.carrera === '' ? "Debe elegir una carrera" : ''
)

const errorCalificacion = computed(() => {
    const c = formulario.calificacion
    if(c === '' || c === null) return "La calificación es obligatoria"
    if(typeof c !== 'number' || Number.isNaN(c)) return "La calificación deber ser un número"
    if(c < 0 || c > 10) return "La calificación debe estar entre 0 y 10"
    return ''
})

const esDuplicado = computed(() => {
    if(formulario.nombre === '' || formulario.carrera === '') return false
    return estudiantes.value.some(
        (e) => e.nombre.toLowerCase() === formulario.nombre.toLowerCase() && e.carrera === formulario.carrera
    )
})

//Lugares disponibles
const lugaresDisponibles = computed(() => Cupo_Maximo - estudiantes.value.length)

const puedeAgregar = computed(
    () => errorNombre.value === '' && errorCarrera.value === '' && errorCalificacion.value === '' && !esDuplicado.value && lugaresDisponibles.value > 0
)

function agregarEstudiante() {
  if (!puedeAgregar.value) return
  estudiantes.value.push({
    id: siguienteId++,
    nombre: formulario.nombre,
    carrera: formulario.carrera,
    calificacion: formulario.calificacion,
  })
  formulario.nombre = ''
  formulario.carrera = ''
  formulario.calificacion = ''
}

//Lista
function claseCalificacion(nota) {
    if(nota >= 6) return 'verde'
    if(nota >= 4) return 'naranja'
    return 'rojo'
}

function eliminar(id) {
    estudiantes.value = estudiantes.value.filter((e) => e.id !== id)
}

function cambiarNota(est, delta) {
    const nueva = est.calificacion + delta
    if (nueva < 0 || nueva > 10) return
    est.calificacion = nueva
}

//Filtros
const filtroCarrera = ref('Todas')
const busqueda = ref('')
const filtroEstado = ref('todos') // todos || aprobados || reprobados

const estudiantesFiltrados = computed(() => {
    const texto = busqueda.value.trim().toLowerCase()
    return estudiantes.value.filter((e) => {
    const Carrera = filtroCarrera.value === 'Todas' || e.carrera === filtroCarrera.value
    const Nombre = e.nombre.toLowerCase().includes(texto)
    const Estado =
      filtroEstado.value === 'todos' ||
      (filtroEstado.value === 'aprobados' && e.calificacion >= 6) ||
      (filtroEstado.value === 'reprobados' && e.calificacion < 6)
    return Carrera && Nombre && Estado
  })
})

//Resumen
const total = computed(() => estudiantes.value.length)

const promedioNumero = computed(() => 
    total.value === 0
    ? 0
    : estudiantes.value.reduce((s, e) => s + e.calificacion, 0) / total.value
)

const promedio = computed(() => (total.value === 0 ? '—' : promedioNumero.value.toFixed(1)))

const aprobados = computed(() => estudiantes.value.filter((e) => e.calificacion >= 6).length)
const reprobados = computed(() => estudiantes.value.filter((e) => e.calificacion < 6).length)

const mejorEstudiante = computed(() => {
  if (estudiantes.value.length === 0) return null
  return estudiantes.value.reduce((mejor, e) =>
    e.calificacion > mejor.calificacion ? e : mejor
  )
})

//Borrar todo
function borrarTodo() {
    estudiantes.value = []
}
</script>


<template>
<main class="contenedor">
<h1>Gestor de Estudiante</h1>

 <section class="resumen">
      <div>Total: <strong>{{ total }}</strong></div>
      <div>
        Promedio:
        <strong :class="{ rojo: total > 0 && promedioNumero < 6 }">{{ promedio }}</strong>
      </div>
      <div>Aprobados: <strong>{{ aprobados }}</strong></div>
      <div>Reprobados: <strong>{{ reprobados }}</strong></div>
      <div>
        Mejor calificación:
        <strong v-if="mejorEstudiante">
          {{ mejorEstudiante.nombre }} ({{ mejorEstudiante.calificacion }})
        </strong>
        <strong v-else>—</strong>
      </div>
      <div>
        Mostrando {{ estudiantesFiltrados.length }} de {{ total }} estudiantes
      </div>
    </section>

    <!--Formulario-->
    <section class="tarjeta">
      <h2>Registrar estudiante</h2>
      <form @submit.prevent="agregarEstudiante">
        <div class="campo">
          <label>Nombre</label>
          <input v-model.trim="formulario.nombre" type="text" placeholder="Nombre" />
          <p v-if="errorNombre" class="error">{{ errorNombre }}</p>
        </div>

        <div class="campo">
          <label>Carrera</label>
          <select v-model="formulario.carrera">
            <option value="" disabled>Selecciona una carrera</option>
            <option v-for="c in carreras" :key="c" :value="c">{{ c }}</option>
          </select>
          <p v-if="errorCarrera" class="error">{{ errorCarrera }}</p>
        </div>

        <div class="campo">
          <label>Calificación (0 a 10)</label>
          <input v-model.number="formulario.calificacion" type="number" step="0.1" />
          <p v-if="errorCalificacion" class="error">{{ errorCalificacion }}</p>
        </div>

        <p v-if="esDuplicado" class="error">Ese estudiante ya existe</p>

        <div class="accion">
          <button type="submit" :disabled="!puedeAgregar">Agregar</button>
          <span v-if="lugaresDisponibles > 0">Quedan {{ lugaresDisponibles }} lugares</span>
          <span v-else class="error">El curso está lleno</span>
        </div>
      </form>
    </section>

    <!--Filtros-->
    <section class="tarjeta filtros">
      <select v-model="filtroCarrera">
        <option value="Todas">Todas</option>
        <option v-for="c in carreras" :key="c" :value="c">{{ c }}</option>
      </select>
      <input v-model="busqueda" type="text" placeholder="Buscar por nombre" />
      <div class="botones">
        <button :class="{ activo: filtroEstado === 'todos' }" @click="filtroEstado = 'todos'">Todos</button>
        <button :class="{ activo: filtroEstado === 'aprobados' }" @click="filtroEstado = 'aprobados'">Aprobados</button>
        <button :class="{ activo: filtroEstado === 'reprobados' }" @click="filtroEstado = 'reprobados'">Reprobados</button>
      </div>
    </section>

    <!--Lista y mensajes -->
     <section class="tarjeta">
      <p v-if="estudiantes.length === 0">Aún no hay estudiantes</p>
      <p v-else-if="estudiantesFiltrados.length === 0">No hay estudiantes con esos filtros</p>
      <ul v-else class="lista">
        <li v-for="est in estudiantesFiltrados" :key="est.id">
          <span class="nombre">
            {{ est.nombre }}<span v-if="est.calificacion === 10"> 👍</span>
          </span>
          <span class="carrera">{{ est.carrera }}</span>
          <span class="nota" :class="claseCalificacion(est.calificacion)">{{ est.calificacion }}</span>
          <span class="botones">
            <button :disabled="est.calificacion <= 0" @click="cambiarNota(est, -1)">−1</button>
            <button :disabled="est.calificacion >= 10" @click="cambiarNota(est, 1)">+1</button>
            <button @click="eliminar(est.id)">✕</button>
          </span>
        </li>
      </ul>
    </section>

    <!--Borrar todo-->
    <button v-show="estudiantes.length > 0" :disabled="estudiantes.length === 0" class="peligro" @click="borrarTodo">
      Borrar todo
    </button>
</main>
</template>


<style scoped>
.contenedor { max-width: 760px; margin: 0 auto; padding: 1rem; text-align: left; }
.resumen { display: grid; grid-template-columns: repeat(auto-fit, minmax(190px, 1fr)); gap: .5rem; background: #87cefa; padding: 1rem; border-radius: 8px; margin-bottom: 1rem; color: #000000 }
.tarjeta { border: 1px solid #ccc; border-radius: 8px; padding: 1rem; margin-bottom: 1rem; background: #000080}
.campo { display: flex; flex-direction: column; margin-bottom: .6rem; }
.campo label { font-weight: 600; margin-bottom: .2rem; }
input, select { padding: .4rem; border: 1px solid #aaa; border-radius: 4px; }
.accion { display: flex; align-items: center; gap: 1rem; }
.filtros { display: flex; flex-wrap: wrap; gap: .6rem; align-items: center; }
.botones { display: flex; gap: .3rem; }
button { padding: .35rem .7rem; border: 1px solid #888; border-radius: 4px; cursor: pointer; background: #fff; color: #222; }
button:disabled { opacity: .45; cursor: not-allowed; }
button.activo { background: #4f46e5; color: #fff; border-color: #4f46e5; }
button.peligro { background: #dc2626; color: #fff; border-color: #dc2626; }
.error { color: #dc2626; margin: .2rem 0; font-size: .9rem; }
.lista { list-style: none; padding: 0; margin: 0; }
.lista li { display: grid; grid-template-columns: 2fr 1.5fr .6fr auto; gap: .5rem; align-items: center; padding: .4rem 0; border-bottom: 1px solid #eee; }
.nota { font-weight: 700; }
.verde { color: #16a34a; }
.naranja { color: #ea580c; }
.rojo { color: #dc2626; }
</style>