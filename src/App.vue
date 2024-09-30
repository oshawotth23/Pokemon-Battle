<template>
  <div id="app">
    <img src="https://upload.wikimedia.org/wikipedia/it/d/de/Pok%C3%A9mon_Battle_Revolution.png" alt="Pokemon Logo" class="logo" />
    
    <div v-if="!gameStarted">
      <h2>Selecciona el número de rondas</h2>
      <button @click="startGame(2)">2 Rondas</button>
      <button @click="startGame(3)">3 Rondas</button>
      <button @click="startGame(4)">4 Rondas</button>
    </div>

    <div v-else>
      <div class="card-container">
        <div class="poke-card">
          <form @submit.prevent="searchPokemon1" class="search-form">
            <input type="text" v-model="pokemonName1" autocomplete="off" placeholder="Busca un Pokémon (Jugador 1)" />
            <button @click.prevent="getRandomPokemon1" class="random-button">Generar Pokémon Aleatorio</button>
          </form>
          <div :class="['poke-card-content', { 'poke-card-expanded': !noResult1 }]">
            <div class="img-container" :style="backgroundStyle1">
              <img class="poke-img" :src="pokeImgSrc1" alt="Pokémon" />
            </div>
            <div v-if="noResult1" class="poke-message">{{ pokeMessage1 }}</div>
            <div v-else class="poke-info">
              <div class="poke-name">Jugador 1: {{ pokeName1 }}</div>
              <div class="poke-id">{{ pokeIdText1 }}</div>
              <div class="poke-type-title">Tipo de Pokémon:</div>
              <div class="poke-types">
                <div v-for="type in pokeTypes1" :key="type" :style="{ backgroundColor: typeColors[type] }" class="poke-type">
                  {{ type }}
                </div>
              </div>
              <select v-model="selectedStat1" class="stat-select" @change="updateStat1">
                <option disabled value="">Selecciona una estadística</option>
                <option v-for="stat in pokeStats" :key="stat">{{ stat }}</option>
              </select>
              <div v-if="selectedStat1" class="stat-display">
                <div class="stat-label">{{ selectedStat1 }}: <span class="stat-value">{{ statValue1 }}</span></div>
                <div class="progress-container">
                  <div class="progress-bar" :style="{ width: (statValue1 / 225) * 100 + '%' }"></div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="poke-card">
          <form @submit.prevent="searchPokemon2" class="search-form">
            <input type="text" v-model="pokemonName2" autocomplete="off" placeholder="Busca un Pokémon (Jugador 2)" />
            <button @click.prevent="getRandomPokemon2" class="random-button">Generar Pokémon Aleatorio</button>
          </form>
          <div :class="['poke-card-content', { 'poke-card-expanded': !noResult2 }]">
            <div class="img-container" :style="backgroundStyle2">
              <img class="poke-img" :src="pokeImgSrc2" alt="Pokémon" />
            </div>
            <div v-if="noResult2" class="poke-message">{{ pokeMessage2 }}</div>
            <div v-else class="poke-info">
              <div class="poke-name">Jugador 2: {{ pokeName2 }}</div>
              <div class="poke-id">{{ pokeIdText2 }}</div>
              <div class="poke-type-title">Tipo de Pokémon:</div>
              <div class="poke-types">
                <div v-for="type in pokeTypes2" :key="type" :style="{ backgroundColor: typeColors[type] }" class="poke-type">
                  {{ type }}
                </div>
              </div>
              <select v-model="selectedStat2" class="stat-select" @change="updateStat2">
                <option disabled value="">Selecciona una estadística</option>
                <option v-for="stat in pokeStats" :key="stat">{{ stat }}</option>
              </select>
              <div v-if="selectedStat2" class="stat-display">
                <div class="stat-label">{{ selectedStat2 }}: <span class="stat-value">{{ statValue2 }}</span></div>
                <div class="progress-container">
                  <div class="progress-bar" :style="{ width: (statValue2 / 225) * 100 + '%' }"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <button @click="battle" class="battle-button" :disabled="!canBattle">A PELEAR</button>
      <div v-if="battleResult" class="battle-result">{{ battleResult }}</div>

      <div v-if="currentRound === totalRounds" class="score-board">
        <h3>Resultado Final</h3>
        <p>Jugador 1 Puntos: {{ player1Score }}</p>
        <p>Jugador 2 Puntos: {{ player2Score }}</p>
        <button @click="resetGame">Regresar a la pantalla de inicio</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
export default {
  data() {
    return {
      gameStarted: false,
      totalRounds: 0,
      currentRound: 0,
      player1Score: 0,
      player2Score: 0,
      
      pokemonName1: '',
      pokeName1: 'Pokedex',
      pokeIdText1: '',
      pokeImgSrc1: 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png',
      pokeTypes1: [],
      pokeMessage1: '',
      noResult1: true,
      selectedStat1: '',
      statValue1: null,

      pokemonName2: '',
      pokeName2: 'Pokedex',
      pokeIdText2: '',
      pokeImgSrc2: 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png',
      pokeTypes2: [],
      pokeMessage2: '',
      noResult2: true,
      selectedStat2: '',
      statValue2: null,

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
      weaknessesMap: {},
      battleResult: '',
    };
  },
  computed: {
    canBattle() {
      return !this.noResult1 && !this.noResult2 && this.selectedStat1 && this.selectedStat2;
    },
    backgroundStyle1() {
      const mainType = this.pokeTypes1[0] || 'default';
      const bgColor = this.typeColors[mainType];
      return {
        background: `radial-gradient(${bgColor} 33%, transparent 33%), rgba(0, 0, 0, 0.1)`,
        backgroundSize: '10px 10px',
        borderRadius: '50%',
        padding: '15px',
      };
    },
    backgroundStyle2() {
      const mainType = this.pokeTypes2[0] || 'default';
      const bgColor = this.typeColors[mainType];
      return {
        background: `radial-gradient(${bgColor} 33%, transparent 33%), rgba(0, 0, 0, 0.1)`,
        backgroundSize: '10px 10px',
        borderRadius: '50%',
        padding: '15px',
      };
    },
  },
  methods: {
    startGame(rounds) {
      this.totalRounds = rounds;
      this.currentRound = 0;
      this.player1Score = 0;
      this.player2Score = 0;
      this.gameStarted = true;
      this.resetPokemonData();
    },
    resetPokemonData() {
      this.pokemonName1 = '';
      this.pokeName1 = 'Pokedex';
      this.pokeIdText1 = '';
      this.pokeImgSrc1 = 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png';
      this.pokeTypes1 = [];
      this.pokeMessage1 = '';
      this.noResult1 = true;
      this.selectedStat1 = '';
      this.statValue1 = null;

      this.pokemonName2 = '';
      this.pokeName2 = 'Pokedex';
      this.pokeIdText2 = '';
      this.pokeImgSrc2 = 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png';
      this.pokeTypes2 = [];
      this.pokeMessage2 = '';
      this.noResult2 = true;
      this.selectedStat2 = '';
      this.statValue2 = null;
    },
    async searchPokemon1() {
      try {
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${this.pokemonName1.toLowerCase()}`);
        if (!response.ok) throw new Error('No se encontró el Pokémon');
        const data = await response.json();
        this.pokeName1 = data.name.charAt(0).toUpperCase() + data.name.slice(1);
        this.pokeIdText1 = `#${data.id}`;
        this.pokeImgSrc1 = data.sprites.front_default;
        this.pokeTypes1 = data.types.map(type => type.type.name);
        this.noResult1 = false;
        this.pokeMessage1 = '';
      } catch (error) {
        this.noResult1 = true;
        this.pokeMessage1 = error.message;
      }
    },
    async getRandomPokemon1() {
      const randomId = Math.floor(Math.random() * 898) + 1;
      const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
      const data = await response.json();
      this.pokeName1 = data.name.charAt(0).toUpperCase() + data.name.slice(1);
      this.pokeIdText1 = `#${data.id}`;
      this.pokeImgSrc1 = data.sprites.front_default;
      this.pokeTypes1 = data.types.map(type => type.type.name);
      this.noResult1 = false;
    },
    async searchPokemon2() {
      try {
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${this.pokemonName2.toLowerCase()}`);
        if (!response.ok) throw new Error('No se encontró el Pokémon');
        const data = await response.json();
        this.pokeName2 = data.name.charAt(0).toUpperCase() + data.name.slice(1);
        this.pokeIdText2 = `#${data.id}`;
        this.pokeImgSrc2 = data.sprites.front_default;
        this.pokeTypes2 = data.types.map(type => type.type.name);
        this.noResult2 = false;
        this.pokeMessage2 = '';
      } catch (error) {
        this.noResult2 = true;
        this.pokeMessage2 = error.message;
      }
    },
    async getRandomPokemon2() {
      const randomId = Math.floor(Math.random() * 898) + 1;
      const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
      const data = await response.json();
      this.pokeName2 = data.name.charAt(0).toUpperCase() + data.name.slice(1);
      this.pokeIdText2 = `#${data.id}`;
      this.pokeImgSrc2 = data.sprites.front_default;
      this.pokeTypes2 = data.types.map(type => type.type.name);
      this.noResult2 = false;
    },
    updateStat1() {
      const stat = this.selectedStat1.toLowerCase();
      this.statValue1 = this.pokeStats.includes(stat) ? Math.floor(Math.random() * 225) : null;
    },
    updateStat2() {
      const stat = this.selectedStat2.toLowerCase();
      this.statValue2 = this.pokeStats.includes(stat) ? Math.floor(Math.random() * 225) : null;
    },
    battle() {
      if (this.statValue1 > this.statValue2) {
        this.battleResult = '¡Jugador 1 gana esta ronda!';
        this.player1Score += 1;
      } else if (this.statValue1 < this.statValue2) {
        this.battleResult = '¡Jugador 2 gana esta ronda!';
        this.player2Score += 1;
      } else {
        this.battleResult = '¡Es un empate!';
      }
      this.currentRound++;
      this.checkGameEnd();
    },
    checkGameEnd() {
      if (this.currentRound >= this.totalRounds) {
        this.battleResult = `¡Juego terminado! Jugador 1: ${this.player1Score}, Jugador 2: ${this.player2Score}`;
      }
    },
    resetGame() {
      this.gameStarted = false;
      this.resetPokemonData();
      this.battleResult = '';
    },
  },
};
</script>

<style scoped>
.logo {
  width: 200px;
}
.card-container {
  display: flex;
  justify-content: space-around;
  margin: 20px;
}
.poke-card {
  border: 2px solid #ccc;
  border-radius: 10px;
  padding: 15px;
  width: 250px;
}
.poke-card-content {
  display: flex;
  flex-direction: column;
}
.poke-name {
  font-size: 20px;
  font-weight: bold;
}
.poke-types {
  display: flex;
  gap: 5px;
}
.poke-type {
  padding: 5px;
  border-radius: 5px;
  color: #fff;
}
.stat-select {
  margin: 10px 0;
}
.progress-container {
  height: 10px;
  background: #eee;
  border-radius: 5px;
  overflow: hidden;
}
.progress-bar {
  height: 100%;
  background: #76c7c0;
}
.battle-button {
  margin: 20px;
  padding: 10px;
  font-size: 16px;
}
.battle-result {
  font-size: 18px;
  margin: 10px 0;
}
.score-board {
  margin-top: 20px;
}
</style>
