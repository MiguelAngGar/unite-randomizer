<script setup>

import { ref, onMounted, nextTick } from 'vue'

const pokemons = ref([])
const pokemonsSelected = ref([])

const equipos = ref([])

var jugadores = ref([

  "Landero",
  "Cobayo",
  "Dustik",
  // "Raser",
  // "Drexzar",
  "Iván",
  ""

])

async function cargarPokemons() {
  try {
    const response = await fetch('/src/assets/pokemon-unite-list.json')
    if (response.ok) {
      const data = await response.json()
      pokemons.value = data
    } else {
      console.error('Error al cargar el archivo JSON')
    }
  } catch (error) {
    console.error('Error al cargar el archivo JSON:', error)
  }
}

onMounted(() => {
  cargarPokemons()
})

function generarPokemon() {
  
  var pokemon = generarPokemonAleatorio()

  while (pokemonsSelected.value.includes(pokemon))
    pokemon = generarPokemonAleatorio()

  return pokemon

}

function generarPokemonAleatorio() {
  let randomIndex = Math.floor(Math.random() * pokemons.value.length)
  let randomPokemon = pokemons.value[randomIndex]
  return randomPokemon
}

function cambioInputGenerado(index) {

  // Se elimina si no hay nada
  if (jugadores.value[index] === '' && (index !== jugadores.value.length - 1 || jugadores.value.length >= 10)) {
    jugadores.value.splice(index, 1)
  }

  // Si escribimos en el ultimo, se añade uno mas
  if (jugadores.value[jugadores.value.length - 1].trim() !== '' && jugadores.value.length < 10) {
    jugadores.value.push('')
  }

}

function generarEquipos() {
  pokemonsSelected.value = []
  equipos.value = []
  const jugadoresFiltrados = jugadores.value.filter(j => j.trim() !== '').sort(() => 0.5 - Math.random())
  const numEquipos = jugadoresFiltrados.length <= 5 ? 2 : Math.ceil(jugadoresFiltrados.length / 5)

  for (let i = 0; i < numEquipos; i++) equipos.value.push({ nombre: `Equipo ${i + 1}`, miembros: [] })

  jugadoresFiltrados.forEach((jugador, i) => {
    let pokemon
    const equipoActual = equipos.value[i % numEquipos].miembros

    // Generar un Pokémon único para el miembro
    do {
      pokemon = generarPokemon()
    } while (equipoActual.some(member => member.pokemon === pokemon))

    equipos.value[i % numEquipos].miembros.push({
      nombre: jugador,
      pokemon: pokemon
    })
  })

  if (jugadoresFiltrados.length <= 5) {
    const [e1, e2] = equipos.value
    const dist = { 5: [3, 2], 4: [2, 2], 3: [2, 1], 2: [1, 1] }
    
    e1.miembros = jugadoresFiltrados.slice(0, dist[jugadoresFiltrados.length][0]).map(j => {
      let pokemon
      do {
        pokemon = generarPokemon()
      } while (e1.miembros.some(member => member.pokemon.name === pokemon.name))
      return { nombre: j, pokemon: pokemon }
    })

    e2.miembros = jugadoresFiltrados.slice(dist[jugadoresFiltrados.length][0]).map(j => {
      let pokemon
      do {
        pokemon = generarPokemon()
      } while (e2.miembros.some(member => member.pokemon.name === pokemon.name))
      return { nombre: j, pokemon: pokemon }
    })
  }
}

function limpiarEquipos() {
  equipos.value = []
}

function refrescarJugador(id, equipoIndex) {
  const equipo = equipos.value[equipoIndex];

  const jugador = equipo.miembros[id];

  let nuevoPokemon;
  do {
    nuevoPokemon = generarPokemon();
  } while (equipo.miembros.some(member => member.pokemon.name === nuevoPokemon.name));

  jugador.pokemon = nuevoPokemon;
}

function obtenerColor(rol) {

  switch (rol) {
    case 'Ofensivo':
      return '#f16c38';

    case 'Agil':
      return '#29a5e3';

    case 'Auxiliar':
      return '#dc923f';

    case 'Equilibrado':
      return '#ce5fd3';

    case 'Defensivo':
      return '#aced5b';
  
    default:
      break;
  }

}


</script>

<template>

  <div>

    <div>
      <div>
        <input 
          v-for="(jugador, index) in jugadores" 
          :key="index" 
          v-model="jugadores[index]" 
          @input="cambioInputGenerado(index)"
          type="text"
          placeholder="Añadir nuevo jugador..." 
        />
      </div>
      <div>
        <button @click="generarEquipos" style="margin-top: 10px">Generar Equipos</button>
        <button @click="limpiarEquipos" style="margin-left: 2px; background-color: indianred;">Limpiar</button>
        <div class="contenedor-equipos">


          <div class="equipo" v-for="(equipo, index) in equipos">


            <div style="display: flex;">
              <div class="titulo-equipo" v-html="equipo.nombre"></div>

              <div style="width: 100%;">

                <div class="miembros-equipo">

                  <div class="jugador-equipo" v-for="jug, id in equipo.miembros">
                    <img :style="{ backgroundColor: obtenerColor(jug.pokemon.role) }" :src="'src/assets/'+jug.pokemon.image" :alt="'Pokemon ' + jug.nombre">
                    <div class="nombre-y-pokemon">
                      <div v-html="jug.pokemon.name"></div>
                      <div class="nombre" v-html="jug.nombre"></div>
                    </div>

                    <div class="boton-refrescar" @click="refrescarJugador(id, index)">
                      <img :src="'src/assets/img/refrescar.png'" alt="refrescar">
                    </div>

                  </div>

                </div>
                
                <div v-if="index < equipos.length - 1" style="display: flex; justify-content: center;">
                  <img :src="'src/assets/img/vs.png'" alt="Vs" style="width: 100px; margin: 30px 0 12px;">
                </div>

              </div>

            </div>

            

          </div>


        </div>
      </div>
    </div>

  </div>

</template>

<style scoped>

.contenedor-equipos {
  margin-top: 30px;
}

.titulo-equipo {
  margin-top: 10px;
  margin-right: -72px;
  font-size: 25px;
  font-weight: 600;
  height: 0px;
  width: 200px;
  transform: rotate(-90deg);
}

.equipo {
  margin-bottom: 10px;
}

.miembros-equipo {
  display: flex;
  flex-direction: c;
  justify-content: space-around;
}

.jugador-equipo {
  box-shadow: 3px 4px 14px 0px rgba(117, 39, 204, 0.4);
  font-size: 15px;
  font-weight: 200;
  width: 250px;
  background-color: #19072d;
  border-radius: 20px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  margin-left: 20px;
  position: relative;
}

.jugador-equipo img {
  border-radius: 12px;
  width: 95%;
  background-color: aliceblue;
  align-self: center;
  margin-top: 8px;
  background-image: url('src/assets/img/square-pattern-30.svg'),linear-gradient(180deg, rgba(0,0,0,0.3), rgba(0,0,0,0) 38%);
  background-size: 200px, auto;
  background-position: center, left top;
}

.nombre-y-pokemon {
  text-align: center;
  padding: 10px 0;
}

.nombre {
  font-size: 25px;
  color: white;
  letter-spacing: 0.65px;
  margin-top: -6px;
}

.boton-refrescar {
  position: absolute;
  width: 38px;
  right: 10px;
  top: 5px;
  cursor: pointer;
}

.boton-refrescar img {
  background: none !important; /* Elimina cualquier fondo */
  box-shadow: none; /* Elimina sombras si es necesario */
  border-radius: 0; /* Si es que otras imágenes tienen bordes redondeados */
}

</style>