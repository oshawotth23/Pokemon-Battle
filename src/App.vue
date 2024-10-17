<template>
  <div id="app">
    <img
      src="https://upload.wikimedia.org/wikipedia/it/d/de/Pok%C3%A9mon_Battle_Revolution.png"
      alt="Logo Pokémon"
      class="logo"
    />

    <div v-if="!juegoIniciado">
      <h2>Elige la cantidad de batallas</h2>
      <button @click="iniciarJuego(2)">2 Batallas</button>
      <button @click="iniciarJuego(3)">3 Batallas</button>
      <button @click="iniciarJuego(4)">4 Batallas</button>
    </div>

    <div v-else>
      <div class="card-container">
        <div class="poke-card">
          <h2 class="player-title">Guerrero 1</h2>
          <div :class="['poke-card-content', { 'poke-card-expanded': !sinResultado1 }]">
            <div class="img-container" :style="fondoStyle1">
              <img class="poke-img" :src="pokeImgSrc1" alt="Pokémon" />
            </div>
            <div v-if="sinResultado1" class="poke-message">{{ pokeMessage1 }}</div>
            <div v-else class="poke-info">
              <div class="poke-name">{{ pokeName1 }}</div>
              <div class="poke-id">{{ pokeIdText1 }}</div>
              <div class="poke-type-title">Tipos del Pokémon:</div>
              <div class="poke-types">
                <div
                  v-for="tipo in pokeTypes1"
                  :key="tipo"
                  :style="{ backgroundColor: typeColors[tipo] }"
                  class="poke-type"
                >
                  {{ tipo }}
                </div>
              </div>
              <select v-model="estadisticaSeleccionada1" class="stat-select" @change="actualizarEstadistica1">
                <option disabled value="">Elige una estadística</option>
                <option v-for="stat in pokeStats" :key="stat">{{ stat }}</option>
              </select>
              <div v-if="estadisticaSeleccionada1" class="stat-display">
                <div class="stat-label" style="font-weight: bold; color: #D5006D;">
                  {{ estadisticaSeleccionada1 }}: 
                  <span class="stat-value">{{ valorEstadistica1 }}</span> / 225
                </div>
                <div class="progress-container">
                  <div class="progress-bar" :style="{ width: (valorEstadistica1 / 225) * 100 + '%' }"></div>
                </div>
              </div>
            </div>
          </div>
          <button 
            @click="buscarPokemonAleatorio1" 
            class="random-button" 
            :disabled="botonAleatorioDeshabilitado1 || juegoTerminado || batallaEnCurso"
            :class="{ 'disabled-button': botonAleatorioDeshabilitado1 || juegoTerminado || batallaEnCurso }"
          >
            Aleatorio
          </button>
        </div>

        <div class="stats-card">
          <h2 class="stats-title">Estadísticas de la Batalla</h2>
          <p>Ronda: {{ rondaActual }} de {{ rondasTotales }}</p>
          <p>Puntos Guerrero 1: {{ puntuacionJugador1 }}</p>
          <p>Puntos Guerrero 2: {{ puntuacionJugador2 }}</p>
          <p v-if="resultadoBatalla" class="battle-result">{{ resultadoBatalla }}</p>
          <button 
            @click="batalla" 
            class="battle-button"
            :disabled="!puedePelear || juegoTerminado"
          >
            ¡A PELEAR!
          </button>
          <button v-if="juegoTerminado" @click="reiniciarJuego" class="reset-button">REINICIAR</button>
        </div>
        <div class="poke-card">
          <h2 class="player-title">Guerrero 2</h2>
          <div :class="['poke-card-content', { 'poke-card-expanded': !sinResultado2 }]">
            <div class="img-container" :style="fondoStyle2">
              <img class="poke-img" :src="pokeImgSrc2" alt="Pokémon" />
            </div>
            <div v-if="sinResultado2" class="poke-message">{{ pokeMessage2 }}</div>
            <div v-else class="poke-info">
              <div class="poke-name">{{ pokeName2 }}</div>
              <div class="poke-id">{{ pokeIdText2 }}</div>
              <div class="poke-type-title">Tipos del Pokémon:</div>
              <div class="poke-types">
                <div
                  v-for="tipo in pokeTypes2"
                  :key="tipo"
                  :style="{ backgroundColor: typeColors[tipo] }"
                  class="poke-type"
                >
                  {{ tipo }}
                </div>
              </div>
              <select v-model="estadisticaSeleccionada2" class="stat-select" @change="actualizarEstadistica2">
                <option disabled value="">Elige una estadística</option>
                <option v-for="stat in pokeStats" :key="stat">{{ stat }}</option>
              </select>
              <div v-if="estadisticaSeleccionada2" class="stat-display">
                <div class="stat-label" style="font-weight: bold; color: #D5006D;">
                  {{ estadisticaSeleccionada2 }}: 
                  <span class="stat-value">{{ valorEstadistica2 }}</span> / 225
                </div>
                <div class="progress-container">
                  <div class="progress-bar" :style="{ width: (valorEstadistica2 / 225) * 100 + '%' }"></div>
                </div>
              </div>
            </div>
          </div>
          <button 
            @click="buscarPokemonAleatorio2" 
            class="random-button" 
            :disabled="botonAleatorioDeshabilitado2 || juegoTerminado || batallaEnCurso"
            :class="{ 'disabled-button': botonAleatorioDeshabilitado2 || juegoTerminado || batallaEnCurso }"
          >
            Aleatorio
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      juegoIniciado: false,
      rondasTotales: 0,
      rondaActual: 0,
      puntuacionJugador1: 0,
      puntuacionJugador2: 0,
      juegoTerminado: false,

      pokeName1: 'Pokedex',
      pokeIdText1: '',
      pokeImgSrc1: 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png',
      pokeTypes1: [],
      pokeMessage1: '',
      sinResultado1: true,
      estadisticaSeleccionada1: '',
      valorEstadistica1: null,

      pokeName2: 'Pokedex',
      pokeIdText2: '',
      pokeImgSrc2: 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png',
      pokeTypes2: [],
      pokeMessage2: '',
      sinResultado2: true,
      estadisticaSeleccionada2: '',
      valorEstadistica2: null,

      pokeStats: ['hp', 'attack', 'defense', 'special-attack', 'special-defense', 'speed'],

      typeColors: {
        electric: '#FFEA70',
        normal: '#B09398',
        fire: '#FF675C',
        water: '#0596C7',
        ice: '#AFEAFD',
        rock: '#999799',
        flying: '#7AE7C7',
        grass: '#4A9681',
        psychic: '#F272B6',
        bug: '#D4E157',
        poison: '#A040A0',
        ground: '#E1C699',
        ghost: '#6D28D9',
        dark: '#5A5A5A',
        dragon: '#DA627D',
        steel: '#1D8A99',
        fighting: '#2F2F2F',
        default: '#2A1A1F',
      },
      resultadoBatalla: '',
      puedePelear: false,
      botonAleatorioDeshabilitado1: false,
      botonAleatorioDeshabilitado2: false,
      batallaEnCurso: false,
    };
  },
  computed: {
    fondoStyle1() {
      const tipoPrincipal = this.pokeTypes1[0] || 'default';
      const colorFondo = this.typeColors[tipoPrincipal];
      return {
        background: `radial-gradient(${colorFondo} 33%, transparent 33%), rgba(0, 0, 0, 0.1)`,
        backgroundSize: '10px 10px',
        borderRadius: '50%',
        padding: '15px',
      };
    },
    fondoStyle2() {
      const tipoPrincipal = this.pokeTypes2[0] || 'default';
      const colorFondo = this.typeColors[tipoPrincipal];
      return {
        background: `radial-gradient(${colorFondo} 33%, transparent 33%), rgba(0, 0, 0, 0.1)`,
        backgroundSize: '10px 10px',
        borderRadius: '50%',
        padding: '15px',
      };
    },
  },
  methods: {
    iniciarJuego(cantidadBatallas) {
      this.juegoIniciado = true;
      this.rondasTotales = cantidadBatallas;
      this.rondaActual = 1;
      this.puntuacionJugador1 = 0;
      this.puntuacionJugador2 = 0;
      this.juegoTerminado = false;
      this.buscarPokemonAleatorio1();
      this.buscarPokemonAleatorio2();
    },
    buscarPokemonAleatorio1() {
      this.sinResultado1 = false;
      this.botonAleatorioDeshabilitado1 = true;
      fetch('https://pokeapi.co/api/v2/pokemon/' + Math.floor(Math.random() * 1000))
        .then(response => response.json())
        .then(data => {
          this.pokeName1 = data.name;
          this.pokeIdText1 = `#${data.id}`;
          this.pokeTypes1 = data.types.map(t => t.type.name);
          this.pokeImgSrc1 = data.sprites.front_default;
          this.botonAleatorioDeshabilitado1 = false;
        })
        .catch(() => {
          this.pokeMessage1 = 'No se encontró Pokémon.';
          this.botonAleatorioDeshabilitado1 = false;
        });
    },
    buscarPokemonAleatorio2() {
      this.sinResultado2 = false;
      this.botonAleatorioDeshabilitado2 = true;
      fetch('https://pokeapi.co/api/v2/pokemon/' + Math.floor(Math.random() * 1000))
        .then(response => response.json())
        .then(data => {
          this.pokeName2 = data.name;
          this.pokeIdText2 = `#${data.id}`;
          this.pokeTypes2 = data.types.map(t => t.type.name);
          this.pokeImgSrc2 = data.sprites.front_default;
          this.botonAleatorioDeshabilitado2 = false;
        })
        .catch(() => {
          this.pokeMessage2 = 'No se encontró Pokémon.';
          this.botonAleatorioDeshabilitado2 = false;
        });
    },
    actualizarEstadistica1() {
      const stat = this.estadisticaSeleccionada1;
      this.valorEstadistica1 = this.pokemon1[stat];
    },
    actualizarEstadistica2() {
      const stat = this.estadisticaSeleccionada2;
      this.valorEstadistica2 = this.pokemon2[stat];
    },
    batalla() {
      this.batallaEnCurso = true;
      // ... código para batallar
    },
    reiniciarJuego() {
      this.juegoIniciado = false;
      this.juegoTerminado = false;
    },
  },
};
</script>

<style>
#app {
  font-family: Arial, sans-serif;
}

.logo {
  width: 200px;
  margin: 0 auto;
  display: block;
}

.card-container {
  display: flex;
  justify-content: space-between;
  margin: 20px;
}

.poke-card {
  flex: 1;
  border: 1px solid #e1e1e1;
  border-radius: 5px;
  padding: 10px;
  margin: 5px;
}

.poke-card-content {
  position: relative;
}

.poke-card-expanded {
  height: 400px;
  transition: all 0.5s ease;
}

.poke-name, .poke-id {
  text-align: center;
  font-weight: bold;
}

.poke-types {
  display: flex;
  justify-content: center;
}

.poke-type {
  padding: 5px;
  margin: 2px;
  border-radius: 5px;
  color: white;
}

.stats-card {
  flex: 1;
  padding: 10px;
  border: 1px solid #e1e1e1;
  border-radius: 5px;
}

.battle-button {
  background-color: #00bcd4;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.reset-button {
  background-color: #e91e63;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.random-button {
  background-color: #9c27b0;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.disabled-button {
  background-color: #ccc;
  cursor: not-allowed;
}

.progress-container {
  width: 100%;
  height: 10px;
  background-color: #e0e0e0;
  border-radius: 5px;
}

.progress-bar {
  height: 100%;
  background-color: #76ff03;
  border-radius: 5px;
}

.stats-title {
  font-size: 18px;
  font-weight: bold;
}

.stats-label {
  color: #ff5722;
}
</style>
