```vue
<script setup>
import { ref, computed, onMounted } from 'vue'

// Variables reactivas
const paises = ref([])
const busqueda = ref('')
const cargando = ref(true)
const error = ref('')

// Consultar la API al abrir la aplicación
onMounted(async () => {
  try {
    const respuesta = await fetch(
      'https://api.restcountries.com/countries/v5?limit=100&pretty=1',
      {
        headers: {
          Authorization: 'Bearer rc_live_5e8054d9460b40c4b4bea4d4e0265845'
        }
      }
    )

    const resultado = await respuesta.json()

   if (!respuesta.ok) {
  console.error(
    'Código HTTP:', respuesta.status,
    'Detalle de la API:', JSON.stringify(resultado, null, 2)
  )

  throw new Error(`Error HTTP: ${respuesta.status}`)
}

    paises.value = resultado.data?.objects ?? []

    if (paises.value.length === 0) {
      error.value = 'La API no devolvió países.'
    }
  } catch (e) {
    console.error('Error al consultar la API:', e)
    error.value =
      'No fue posible cargar los países. Revisa la consola.'
  } finally {
    cargando.value = false
  }
})

// Filtrar países por nombre o región
const paisesFiltrados = computed(() => {
  const texto = busqueda.value.trim().toLowerCase()

  return paises.value.filter((pais) => {
    const nombre = pais.names?.common ?? ''
    const region = pais.region ?? ''

    return (
      nombre.toLowerCase().includes(texto) ||
      region.toLowerCase().includes(texto)
    )
  })
})
</script>

<template>
  <main class="contenedor">
    <header class="encabezado">
      <h1>🌎 Explorador de Países</h1>
      <p>Descubre información de países de todo el mundo.</p>
    </header>

    <input
      v-model="busqueda"
      type="text"
      placeholder="Buscar país o región..."
      class="buscador"
      aria-label="Buscar país o región"
    />

    <p v-if="cargando" class="mensaje">
      ⏳ Cargando países...
    </p>

    <p v-else-if="error" class="mensaje error">
      {{ error }}
    </p>

    <p
      v-else-if="paisesFiltrados.length === 0"
      class="mensaje"
    >
      No se encontraron países.
    </p>

    <section v-else class="rejilla">
      <article
        v-for="pais in paisesFiltrados"
        :key="pais.codes?.alpha_3 || pais.names?.common"
        class="tarjeta"
      >
        <img
          v-if="pais.codes?.alpha_2"
          :src="`https://flags.restcountries.com/v5/w320/${pais.codes.alpha_2.toLowerCase()}.png`"
          :alt="`Bandera de ${pais.names?.common || 'país'}`"
        />

        <h2>
          {{ pais.names?.common || 'Sin nombre' }}
        </h2>

        <p>
          <strong>Capital:</strong>
          {{ pais.capitals?.join(', ') || 'No disponible' }}
        </p>

        <p>
          <strong>Región:</strong>
          {{ pais.region || 'No disponible' }}
        </p>

        <p>
          <strong>Población:</strong>
          {{
            typeof pais.population === 'number'
              ? pais.population.toLocaleString('es-MX')
              : 'No disponible'
          }}
        </p>
      </article>
    </section>

    <footer>
      Información consultada mediante REST Countries API.
    </footer>
  </main>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, Helvetica, sans-serif;
  background: #f1f5f9;
  color: #1e293b;
}

.contenedor {
  max-width: 1100px;
  margin: 0 auto;
  padding: 30px 20px;
}

.encabezado {
  text-align: center;
  padding: 30px 20px;
  background: #0f766e;
  color: white;
  border-radius: 15px;
  margin-bottom: 25px;
}

.encabezado h1 {
  font-size: 32px;
  margin-top: 0;
}

.encabezado p {
  margin-bottom: 0;
}

.buscador {
  width: 100%;
  padding: 14px;
  margin-bottom: 25px;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  font-size: 16px;
  outline-color: #0f766e;
}

.rejilla {
  display: grid;
  grid-template-columns: repeat(
    auto-fit,
    minmax(230px, 1fr)
  );
  gap: 20px;
}

.tarjeta {
  padding: 18px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px #00000012;
  overflow-wrap: anywhere;
}

.tarjeta img {
  display: block;
  width: 100%;
  height: 130px;
  object-fit: contain;
  margin-bottom: 15px;
}

.tarjeta h2 {
  color: #0f766e;
  font-size: 21px;
}

.tarjeta p {
  line-height: 1.5;
}

.mensaje {
  padding: 20px;
  text-align: center;
  background: white;
  border-radius: 10px;
}

.error {
  color: #b91c1c;
}

footer {
  margin-top: 30px;
  padding: 15px;
  text-align: center;
  color: #64748b;
  font-size: 14px;
}

@media (max-width: 600px) {
  .contenedor {
    padding: 15px;
  }

  .encabezado h1 {
    font-size: 25px;
  }
}
</style>
```