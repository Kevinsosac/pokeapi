<template>
  <div class="container">
    <!-- Modal -->
    <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-xl">
        <div class="modal-content">
          <div class="modal-header">
            <h1 class="modal-title fs-5" id="exampleModalLabel"></h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <p id="nombre">{{ pokemonSeleccionado ? capitalizeFirstLetter(pokemonSeleccionado.nombre) : '' }}</p>
            <div class="container-modal-content">
              <div class="container-imagen-poke">
                <div class="imagenpoke">
                  <img :src="pokemonSeleccionado ? pokemonSeleccionado.imagen : ''" alt="Imagen del Pokémon">
                </div>
              </div>
            </div>
            

            <div class="container-debi-type">
              <div v-if="pokemonSeleccionado">
                <!-- Tipo -->
                <h1>Tipo</h1>
                <div v-for="(tipo, i) in pokemonSeleccionado.tipos" :key="i">
                  <button :class="['btn-tipo', getTipoClase(tipo.name).clase]">
                    <i :class="[getTipoClase(tipo.name).icono]"></i>
                    {{ capitalizeFirstLetter(tipo.name) }}
                  </button>
                </div>
                
              </div>
            </div>
            <section class="habi">
              <div class="container-skills">
              <div class="skill">
                <p><span>HP</span>{{pokemonSeleccionado ? pokemonSeleccionado.hp :  ''}}</p>
                <div class="progress" :style="{ '--wth': pokemonSeleccionado ? pokemonSeleccionado.hp + 'px' : 0 }"></div>
              </div>
              <div class="skill">
                <p><span>Attack</span>{{pokemonSeleccionado ? pokemonSeleccionado.attack : ''}}</p>
                <div class="progress" :style="{ '--wth': pokemonSeleccionado ? pokemonSeleccionado.attack + 'px' : 0 }"></div>
              </div>
              <div class="skill">
                <p><span>Defense</span>{{pokemonSeleccionado ? pokemonSeleccionado.defense : ''}}</p>
                <div class="progress" :style="{ '--wth': pokemonSeleccionado ? pokemonSeleccionado.defense  + 'px' : 0 }"></div>
              </div>
              <div class="skill">
                <p><span>Special Attack</span>{{pokemonSeleccionado ? pokemonSeleccionado.special_attack   : ''}}</p>
                <div class="progress" :style="{ '--wth': pokemonSeleccionado ? pokemonSeleccionado.special_attack + 'px': 0 }"></div>
              </div>
              <div class="skill">
                <p><span>Special Defense</span>{{pokemonSeleccionado ? pokemonSeleccionado.special_defense  : ''}}</p>
                <div class="progress" :style="{ '--wth': pokemonSeleccionado ? pokemonSeleccionado.special_defense  + 'px' : 0 }"></div>
              </div>
              <div class="skill">
                <p><span>Speed</span>{{pokemonSeleccionado ? pokemonSeleccionado.speed : ''}}</p>
                <div class="progress" :style="{ '--wth': pokemonSeleccionado ? pokemonSeleccionado.speed  + 'px': 0 }"></div>
              </div>
            </div>
            </section>
          </div>
          <div class="modal-footer">
          </div>
        </div>
      </div>
    </div>
    <!-- Pokedex -->
    <header>
      <div class="titulo">
          <a href="#" class="pokedex"><img src="/src/assets/Pokedex.png" alt="" data-v-7a7a37b1="">
          </a>
        </div>
    </header>
    <div class="busquedas">
  <div class="filtro">
  <label for="tipoFiltro" class="visually-hidden">Selecciona un tipo:</label>
  <select v-model="tipoFiltro" id="tipoFiltro" class="form-select" @change="filtrarPorTipo">
    <option value="">Todos los tipos</option> <!-- Nueva opción -->
    <option v-for="tipo in tiposDisponibles" :key="tipo" :value="tipo">
      {{ capitalizeFirstLetter(tipo) }}
    </option>
  </select>
</div>
  <div class="input-pokemon">
    <input v-model="searchTerm" type="text" placeholder="Escribe el nombre del Pokémon">
    <button @click="buscarPokemon" type="button" class="btn btn-primary">
      <i class="fa-solid fa-magnifying-glass"></i>
    </button>
  </div>
</div>
    <div class="container-cards">
      <div v-if="pokemonsFiltrados.length === 0">
    <p>No hay Pokémon del tipo seleccionado.</p>
  </div>
  <div class="card" v-for="(pokemon, index) in pokemonsFiltrados" :key="index">
          <div class="card">
  <div class="card__content">
    <p class="card__title">
    </p><p class="card__description"></p>
  </div>
</div>
        <div class="imagen-fondo">
          <button class="btn-poke" @click="seleccionarPokemon(pokemon)" data-bs-toggle="modal" data-bs-target="#exampleModal"><img :src="pokemon.imagen" alt=""></button>
        </div>
        <div class="info">
          <h1>{{ capitalizeFirstLetter(pokemon.nombre) }}</h1>
          <h2># {{ pokemon.numero }}</h2>
        </div>
        <div class="container-tipos">
          <div v-for="(item, i) in pokemon.tipos" :key="i">
            <button :class="['btn-tipo', getTipoClase(pokemon.tipos[i].name).clase]">
              <i :class="[getTipoClase(pokemon.tipos[i].name).icono]"></i>
              {{ capitalizeFirstLetter(item.name) }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <button type="button" class="btn btn-warning" @click="mostrarMas">Mostrar más pokemones</button>
    <footer>
    </footer>
  </div>
</template>

<script setup>
import axios from 'axios'
import { ref, onMounted, computed, watch } from 'vue';

let pokemons = ref([]);
let pokemonsFiltrados = ref([]);
let searchTerm = ref('');
let link = ref('https://pokeapi.co/api/v2/pokemon?limit=50&offset=0');
let tipoFiltro = ref('');

// Calcula los tipos disponibles en todos los Pokémon
const tiposDisponibles = computed(() => {
  let tipos = new Set([]); // Agrega la opción "Todos los tipos"
  pokemons.value.forEach(pokemon => {
    pokemon.tipos.forEach(tipo => tipos.add(tipo.name));
  });
  return Array.from(tipos);
});

// Función para filtrar los Pokémon por tipo
const filtrarPorTipo = () => {
  if (tipoFiltro.value === '') {
    // Si no hay un tipo seleccionado, muestra todos los Pokémon
    pokemonsFiltrados.value = pokemons.value;
  } else {
    // Filtra los Pokémon por el tipo seleccionado
    pokemonsFiltrados.value = pokemons.value.filter(pokemon =>
      pokemon.tipos.some(tipo => tipo.name === tipoFiltro.value)
    );
  }
};
async function obtenerPokemon(url) {
  console.log("h");
  try {
    let r = await axios.get(url);
    console.log(r);

    const tipos = ref([]);
    const debilidades = ref([])

    for (let i = 0; i < r.data.types.length; i++) {
      const tipoActual = r.data.types[i].type;
      const tipopoke = r.data.types[i].type.name.toLowerCase(); 

      tipos.value.push(tipoActual);

      let urldebi = tipoActual.url; 

      let debilidad = await axios.get(urldebi);

      for (let j = 0; j < debilidad.data.damage_relations.double_damage_from.length; j++) {
        const debilidadActual = debilidad.data.damage_relations.double_damage_from[j].name.toLowerCase();


        if (debilidadActual !== tipopoke && !debilidades.value.includes(debilidadActual)) {
          debilidades.value.push(debilidadActual);
        }
      }
    }
    pokemons.value.push({
      imagen: r.data.sprites.other['official-artwork'].front_default,
      nombre: r.data.name,
      numero: r.data.id,
      tipos: tipos,
      debilidades: debilidades,
      hp: r.data.stats[0].base_stat,
      attack: r.data.stats[1].base_stat,
      defense: r.data.stats[2].base_stat,
      special_attack: r.data.stats[3].base_stat,
      special_defense: r.data.stats[4].base_stat,
      speed: r.data.stats[5].base_stat,
    });
  } catch (error) {
    console.error("Error al obtener los datos de un Pokémon:", error);
  }
}



function getTipoClase(tipo) {
  const clasesPorTipo = {
    water: { clase: "tipo-agua"},
    fire: { clase: "tipo-fuego"},
    grass: { clase: "tipo-planta"},
    electric: { clase: "tipo-electrico"},
    ice: { clase: "tipo-hielo"},
    fighting: { clase: "tipo-lucha"},
    poison: { clase: "tipo-veneno"},
    ground: { clase: "tipo-tierra"},
    flying: { clase: "tipo-volador"},
    psychic: { clase: "tipo-psiquico"},
    bug: { clase: "tipo-bicho" },
    rock: { clase: "tipo-roca"},
    ghost: { clase: "tipo-fantasma"},
    dragon: { clase: "tipo-dragon"},
    dark: { clase: "tipo-siniestro"},
    steel: { clase: "tipo-acero"},
    fairy: { clase: "tipo-hada"},
    normal: { clase: "tipo-normal"},
  };

  return clasesPorTipo[tipo] || "tipo-desconocido";
}

async function obtenerPokemons() {
  let pokemonsResponse = await axios.get("https://pokeapi.co/api/v2/pokemon?limit=50&offset=0");
  let pokemonUrls = pokemonsResponse.data.results.map((pokemon) => pokemon.url);

  for (const url of pokemonUrls) {
    await obtenerPokemon(url);
  }
}

function capitalizeFirstLetter(str) {
  return str ? str.charAt(0).toUpperCase() + str.slice(1) : "";
}

let pokemonSeleccionado = ref(null); 

function seleccionarPokemon(pokemon) {
  pokemonSeleccionado.value = pokemon; 
}



const buscarPokemon = () => {
  const busqueda = searchTerm.value.toLowerCase().trim();

  if (busqueda === '') {
    // Si la búsqueda está vacía, mostrar todos los Pokémon
    pokemonsFiltrados.value = pokemons.value;
  } else {
    // Crear un objeto para realizar el filtrado único por número de Pokémon
    const pokemonsUnicos = {};
    
    // Filtrar los Pokémon por nombre y agregarlos al objeto de Pokémon únicos
    pokemons.value.filter(pokemon =>
      pokemon.nombre.toLowerCase().includes(busqueda)
    ).forEach(pokemon => {
      pokemonsUnicos[pokemon.numero] = pokemon;
    });

    // Convertir el objeto de Pokémon únicos de nuevo a un array
    pokemonsFiltrados.value = Object.values(pokemonsUnicos);
  }
};

const mostrarMas = async () => {
  try {console.log(link.value);
    let pokemonsResponse = await axios.get(link.value);
    let pokemonUrls = pokemonsResponse.data.results.map((pokemon) => pokemon.url);

    for (const url of pokemonUrls) {
      await obtenerPokemon(url);
    }

    // Actualizar el enlace para cargar los siguientes 50 Pokémon
    link.value = pokemonsResponse.data.next;

    // Actualizar la lista de pokemonsFiltrados después de cargar más Pokémon
    filtrarPorTipo();
  } catch (error) {
    console.error('Error al cargar más Pokémon:', error);
  }
};
onMounted(() => {
  watch(tipoFiltro, filtrarPorTipo);
});


watch(tipoFiltro, filtrarPorTipo);


</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Martian+Mono:wght@400;500;600&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Dela+Gothic+One&display=swap');



#nombre{
  font-size: 45px;
}

.modal-body {
    position: relative;
    flex: 1 1 auto;
    padding: var(--bs-modal-padding);
    display: flex;
}

.container{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  align-content: center;
  width: 100%;
  background-image: url('pika.png');
}

.container-skills .skill{
  margin: 20px 0;
}

.container-skills .skill p{
  width: 300px;
  display: flex;
  justify-content: space-between;
}

.container-skills .skill .progress{
  position: relative;
  width: 100%;
  height: 6px;
  background: #999;
  border-radius:6px ;
  overflow: hidden;
  margin: 5px 0;
}

.container-skills .skill .progress::before{
  content:'' ;
  position: absolute;
  width: var(--wth, 0px); 
  height: 100%;
  background: #0280fe;
}

header{
  width: 100%;
  height: 200px;
  display: flex;
  align-items: center;
}
.filtro button{
  border: none;
  height: 50px;
  width: auto;
  font-size: 20px;
  display: flex;
  align-items: center;
  gap: 4px;
}
.titulo{
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: space-evenly;
}
.pokeball-izq{
  height: 100px;
  width: 100px;
  transform: scaleX(-1)
}
.pokeball-der{
  height: 100px;
  width: 100px;
}

.card {
  position: relative;
  width: 300px;
  height: 200px;
  background: linear-gradient(-45deg, #be9d75 0%, #676fe0 100% );
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  transition: all 0.6s cubic-bezier(0.23, 1, 0.320, 1);
}

.card svg {
  width: 48px;
  fill: #333;
  transition: all 0.6s cubic-bezier(0.23, 1, 0.320, 1);
}

.card:hover {
  transform: rotate(-5deg) scale(1.1);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
}

.card__content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(-45deg);
  width: 100%;
  height: 100%;
  padding: 20px;
  box-sizing: border-box;
  background-color: #fff;
  opacity: 0;
  transition: all 0.6s cubic-bezier(0.23, 1, 0.320, 1);
}

.card:hover .card__content {
  transform: translate(-50%, -50%) rotate(0deg);
  opacity: 1;
}

.card__title {
  margin: 0;
  font-size: 24px;
  color: #333;
  font-weight: 700;
}

.card__description {
  margin: 10px 0 0;
  font-size: 14px;
  color: #777;
  line-height: 1.4;
}

.card:hover svg {
  scale: 0;
  transform: rotate(-45deg);
}

.input-pokemon{
  display: flex;
  justify-content: flex-end;
}
.input-pokemon input{
  width: 250px;
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
  border: 1 px solid blue;
}
.input-pokemon button{
  border-top-left-radius: 0px;
  border-bottom-left-radius: 0px;
}
.busquedas{
  display: flex;
  width: 50%;
  justify-content: space-evenly;
}
.pikas{
  height: 120px;
  width: 100%;
  display: flex;
  justify-content:space-between ;
}
.pikas img{
  height: 100%;

}
.pika-der{
  transform: scaleX(-1)
}
.container-cards{
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-wrap: wrap;
  /* background-color: white; */
  width: 100%;
}
.btn-poke {
  background: none;
  border: none;
  width: 100%;
}
.btn-poke img{
  height: 100%;
  width: 100%;
  background-color: rgba(128, 128, 128, 0.088);
  border-radius: 10px;
}
.card{
  width: 300px;
  height: 460px;
  margin: 10px;
  border: none ;
}
.info{
  text-align: center;
  font-family: 'Dela Gothic One', cursive;
}
.info h2{
  margin: 8px;
  font-size: 20px;

}
.info h1{
  margin: 8px;
  font-size: 25px;
}
.container-tipos{
  display: flex;
  width: 100%;
  justify-content: space-evenly;
  flex-wrap: wrap;

}
.btn-tipo{
  border: none;
  height: 40px;
  width: 54px;
  display: flex;
  justify-content: space-around;
  align-items: center;
  border-radius: 10px;
  margin: 10px;
}
.tipo-agua{
  background-color: hsla(240, 100%, 45%, 0.501);
}
.tipo-fuego{
  background-color: rgba(255, 166, 0, 0.497);
}
.tipo-planta{
  background-color: rgba(0, 128, 0, 0.504);
}
.tipo-electrico{
  background-color: rgba(255, 255, 0, 0.486);
}
.tipo-hielo{
  background-color: rgba(0, 255, 255, 0.492);
}
.tipo-lucha{
  background-color: rgba(255, 0, 0, 0.504);
}
.tipo-planta{
  background-color: rgba(0, 128, 0, 0.504);
}
.tipo-veneno{
  background-color: rgba(128, 0, 128, 0.475);
}
.tipo-tierra{
  background-color: rgba(165, 42, 42, 0.505);
}
.tipo-volador{
  background-color: rgba(250, 221, 162, 0.46);
}
.tipo-psiquico{
  background-color: rgba(238, 120, 140, 0.481);
}
.tipo-bicho{
  background-color: rgba(122, 165, 42, 0.505);
}
.tipo-roca{
  background-color: rgba(128, 128, 128, 0.533);
}
.tipo-fantasma{
  background-color: rgba(95, 13, 95, 0.477);
}
.tipo-dragon{
  background-color: rgba(71, 5, 5, 0.505);
}
.tipo-siniestro{
  background-color: rgba(0, 0, 0, 0.455);
}
.tipo-acero{
  background-color: rgba(85, 107, 47, 0.453);
}
.tipo-hada{
  background-color: rgba(165, 42, 42, 0.505);
}
.tipo-normal{
  background-color: rgba(97, 97, 238, 0.456);
}
.btn-warning{
  margin: 10px;
}
.container-debi.type{
  margin-left: 30%;
}
.container-imagen-poke{
  height: 270px;
  width: 270px;
  background-color: white;
  display: flex;
  margin-left: -52%;
  justify-content: center;
  align-items: center;
  -webkit-box-shadow: 2px 2px 10px -3px rgba(0,0,0,0.57);
  -moz-box-shadow: 2px 2px 10px -3px rgba(0,0,0,0.57);
  box-shadow: 2px 2px 10px -3px rgba(0,0,0,0.57);
  margin-top: 50%;
}
.imagenpoke{
  background-color: rgba(128, 128, 128, 0.088);
  height: 250px;
  width: 250px;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
}
.imagenpoke img{
  height: 350px;
}
.container-debi-type{
     margin-top: 38%;
    margin-left: -26%;
    display: flex;
}
.esquina-arriba{
  position: absolute;
  top: -0.3%;
  left: -0.3%;
  height: 100px;
  width: 100px;
  z-index: 20;
}
.esquina-abajo{
  position: absolute;
  height: 100px;
  width: 100px;
  transform: rotate(-180deg);
  z-index: 50;
  top: 14.2%;
  left: 18.2%;
}



@media screen and (max-width: 1200px){
 .esquina-arriba{
  left: -0.4%;
 }
 .esquina-abajo{
  left: 26%;
 }
}
@media screen and (max-width: 991px){
 .esquina-arriba{
  left: -0.5%;
 }
 .esquina-abajo{
  left: 41.1%;
  top: 14.3%;

 }
}
  
.esquina-arriba img, .esquina-abajo img{
  height: 100%;
  width: 100%;
}

.habi{
  width: 600px;
  margin: auto;
  padding: 10px;
}

.contai{
  width: 100%;
  background-color: #888;
  border-radius: 4rem;
}

.porcen{
  text-align: right;
  padding: 10px;
  color: white;
  font-weight: 900;
}

.Hp{
  background-color: grenn;
  animation: hp 2s forwards;
  border-radius:4rem ;
}
@keyframes hp{
  0%{
    width: 0;
  }
  100%{
    width: 90%;
  }
}
</style>