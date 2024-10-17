<template>
  <div id="app">
    <img
      src="https://upload.wikimedia.org/wikipedia/it/d/de/Pok%C3%A9mon_Battle_Revolution.png"
      alt="Logo Pokémon"
      class="logo"
    />

    <div v-if="!gameStarted">
      <h2>Elige la cantidad de batallas</h2>
      <button @click="startGame(2)">2 Batallas</button>
      <button @click="startGame(3)">3 Batallas</button>
      <button @click="startGame(4)">4 Batallas</button>
    </div>

    <div v-else>
      <div class="card-container">
        <div class="poke-card">
          <h2 class="player-title">Guerrero 1</h2>
          <div :class="['poke-card-content', { 'poke-card-expanded': !noResult1 }]">
            <div class="img-container" :style="backgroundStyle1">
              <img class="poke-img" :src="pokeImgSrc1" alt="Pokémon" />
            </div>
            <div v-if="noResult1" class="poke-message">{{ pokeMessage1 }}</div>
            <div v-else class="poke-info">
              <div class="poke-name">{{ pokeName1 }}</div>
              <div class="poke-id">{{ pokeIdText1 }}</div>
              <div class="poke-type-title">Elementos del Pokémon:</div>
              <div class="poke-types">
                <div
                  v-for="type in pokeTypes1"
                  :key="type"
                  :style="{ backgroundColor: typeColors[type] }"
                  class="poke-type"
                >
                  {{ type }}
                </div>
              </div>
              <select v-model="selectedStat1" class="stat-select" @change="updateStat1">
                <option disabled value="">Elige una estadística</option>
                <option v-for="stat in pokeStats" :key="stat">{{ stat }}</option>
              </select>
              <div v-if="selectedStat1" class="stat-display">
                <div class="stat-label" style="font-weight: bold; color: #D5006D;">
                  {{ selectedStat1 }}: 
                  <span class="stat-value">{{ statValue1 }}</span> / 225
                </div>
                <div class="progress-container">
                  <div class="progress-bar" :style="{ width: (statValue1 / 225) * 100 + '%' }"></div>
                </div>
              </div>
            </div>
          </div>
          <button 
            @click="searchRandomPokemon1" 
            class="random-button" 
            :disabled="randomButtonDisabled1 || gameOver || battleInProgress"
            :class="{ 'disabled-button': randomButtonDisabled1 || gameOver || battleInProgress }"
          >
            Aleatorio
          </button>
        </div>

        <div class="stats-card">
          <h2 class="stats-title">Estadísticas de la Batalla</h2>
          <p>Ronda: {{ currentRound }} de {{ totalRounds }}</p>
          <p>Puntos Guerrero 1: {{ player1Score }}</p>
          <p>Puntos Guerrero 2: {{ player2Score }}</p>
          <p v-if="battleResult" class="battle-result">{{ battleResult }}</p>
          <button 
            @click="battle" 
            class="battle-button"
            :disabled="!canBattle || gameOver"
          >
            ¡A PELEAR!
          </button>
          <button v-if="gameOver" @click="resetGame" class="reset-button">REINICIAR</button>
        </div>
        <div class="poke-card">
          <h2 class="player-title">Guerrero 2</h2>
          <div :class="['poke-card-content', { 'poke-card-expanded': !noResult2 }]">
            <div class="img-container" :style="backgroundStyle2">
              <img class="poke-img" :src="pokeImgSrc2" alt="Pokémon" />
            </div>
            <div v-if="noResult2" class="poke-message">{{ pokeMessage2 }}</div>
            <div v-else class="poke-info">
              <div class="poke-name">{{ pokeName2 }}</div>
              <div class="poke-id">{{ pokeIdText2 }}</div>
              <div class="poke-type-title">Elementos del Pokémon:</div>
              <div class="poke-types">
                <div
                  v-for="type in pokeTypes2"
                  :key="type"
                  :style="{ backgroundColor: typeColors[type] }"
                  class="poke-type"
                >
                  {{ type }}
                </div>
              </div>
              <select v-model="selectedStat2" class="stat-select" @change="updateStat2">
                <option disabled value="">Elige una estadística</option>
                <option v-for="stat in pokeStats" :key="stat">{{ stat }}</option>
              </select>
              <div v-if="selectedStat2" class="stat-display">
                <div class="stat-label" style="font-weight: bold; color: #D5006D;">
                  {{ selectedStat2 }}: 
                  <span class="stat-value">{{ statValue2 }}</span> / 225
                </div>
                <div class="progress-container">
                  <div class="progress-bar" :style="{ width: (statValue2 / 225) * 100 + '%' }"></div>
                </div>
              </div>
            </div>
          </div>
          <button 
            @click="searchRandomPokemon2" 
            class="random-button" 
            :disabled="randomButtonDisabled2 || gameOver || battleInProgress"
            :class="{ 'disabled-button': randomButtonDisabled2 || gameOver || battleInProgress }"
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
      gameStarted: false,
      totalRounds: 0,
      currentRound: 0,
      player1Score: 0,
      player2Score: 0,
      gameOver: false,

      pokeName1: 'Pokedex',
      pokeIdText1: '',
      pokeImgSrc1: 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png',
      pokeTypes1: [],
      pokeMessage1: '',
      noResult1: true,
      selectedStat1: '',
      statValue1: null,

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
      battleResult: '',
      canBattle: false,
      randomButtonDisabled1: false,
      randomButtonDisabled2: false,
      battleInProgress: false,
    };
  },
  computed: {
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
      this.gameStarted = true;
      this.totalRounds = rounds;
      this.currentRound = 1;
      this.player1Score = 0;
      this.player2Score = 0;
      this.gameOver = false;
    },

    async searchRandomPokemon1() {
      this.randomButtonDisabled1 = true;
      try {
        const randomId = Math.floor(Math.random() * 898) + 1;
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
        const data = await response.json();
        this.pokeName1 = data.name;
        this.pokeIdText1 = `ID: ${data.id}`;
        this.pokeImgSrc1 = data.sprites.other['official-artwork'].front_default;
        this.pokeTypes1 = data.types.map((type) => type.type.name);
        this.noResult1 = false;
        this.pokeMessage1 = '';

        if (this.selectedStat1) {
          this.statValue1 = data.stats.find(stat => stat.stat.name === this.selectedStat1).base_stat;
        } else {
          this.statValue1 = Math.floor(Math.random() * 226);
        }
        
      } catch (error) {
        this.noResult1 = true;
        this.pokeMessage1 = 'Pokémon no encontrado';
      } finally {
        this.canBattle = this.selectedStat1 && this.selectedStat2;
        this.randomButtonDisabled1 = false;
      }
    },

    async searchRandomPokemon2() {
      this.randomButtonDisabled2 = true;
      try {
        const randomId = Math.floor(Math.random() * 898) + 1;
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
        const data = await response.json();
        this.pokeName2 = data.name;
        this.pokeIdText2 = `ID: ${data.id}`;
        this.pokeImgSrc2 = data.sprites.other['official-artwork'].front_default;
        this.pokeTypes2 = data.types.map((type) => type.type.name);
        this.noResult2 = false;
        this.pokeMessage2 = '';

        if (this.selectedStat2) {
          this.statValue2 = data.stats.find(stat => stat.stat.name === this.selectedStat2).base_stat;
        } else {
          this.statValue2 = Math.floor(Math.random() * 226);
        }
        
      } catch (error) {
        this.noResult2 = true;
        this.pokeMessage2 = 'Pokémon no encontrado';
      } finally {
        this.canBattle = this.selectedStat1 && this.selectedStat2;
        this.randomButtonDisabled2 = false;
      }
    },

    updateStat1() {
      if (this.selectedStat1) {
        this.statValue1 = Math.floor(Math.random() * 226);
      }
      this.canBattle = this.selectedStat1 && this.selectedStat2;
    },
    
    updateStat2() {
      if (this.selectedStat2) {
        this.statValue2 = Math.floor(Math.random() * 226);
      }
      this.canBattle = this.selectedStat1 && this.selectedStat2;
    },

    async battle() {
      if (this.selectedStat1 && this.selectedStat2) {
        this.battleInProgress = true;

        if (this.statValue1 > this.statValue2) {
          this.player1Score++;
          this.battleResult = '¡Guerrero 1 triunfa en esta batalla!';
        } else if (this.statValue2 > this.statValue1) {
          this.player2Score++;
          this.battleResult = '¡Guerrero 2 triunfa en esta batalla!';
        } else {
          this.battleResult = '¡Un empate épico!';
        }

        this.currentRound++;
        if (this.currentRound > this.totalRounds) {
          this.gameOver = true;
          this.battleResult = this.player1Score > this.player2Score ? '¡Guerrero 1 se lleva el juego!' : 
                             this.player2Score > this.player1Score ? '¡Guerrero 2 se lleva el juego!' : '¡Un empate en el juego!';
        }

        this.battleInProgress = false;
      }
    },

    resetGame() {
      this.gameStarted = false;
      this.pokeName1 = '';
      this.pokeName2 = '';
      this.noResult1 = true;
      this.noResult2 = true;
      this.statValue1 = null;
      this.statValue2 = null;
      this.selectedStat1 = '';
      this.selectedStat2 = '';
      this.battleResult = '';
      this.player1Score = 0;
      this.player2Score = 0;
      this.currentRound = 0;
      this.gameOver = false;
      this.randomButtonDisabled1 = false;
      this.randomButtonDisabled2 = false;
    }
  },
};
</script>

<style>
body {
  font-family: 'Courier New', Courier, monospace;
  background: url('https://pbs.twimg.com/media/GW5sfaPW4AAkiph?format=jpg&name=large') no-repeat center center fixed;
  background-size: cover;
  text-align: center;
  padding: 20px;
}

.logo {
  width: 350px;
  margin-bottom: 20px;
}

.card-container {
  display: flex;
  justify-content: space-around;
  margin-top: 20px;
  flex-wrap: wrap;
}

.poke-card {
  background: #fff;
  border: 2px dashed #ff69b4;
  border-radius: 15px;
  width: 300px;
  padding: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s;
}

.poke-card:hover {
  transform: scale(1.05);
}

.player-title {
  font-size: 26px;
  color: #8A2BE2;
  margin: 10px 0;
}

.poke-card-content {
  transition: all 0.3s ease;
}

.poke-card-expanded {
  height: auto;
}

.img-container {
  margin-bottom: 10px;
}

.poke-img {
  width: 100%;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

.poke-message {
  color: #ff4500;
}

.poke-info {
  margin-top: 10px;
}

.poke-name {
  font-weight: bold;
  font-size: 22px;
}

.poke-id, .poke-type-title {
  font-size: 14px;
  color: #555;
}

.poke-types {
  display: flex;
  justify-content: center;
  margin: 5px 0;
}

.poke-type {
  padding: 5px 10px;
  border-radius: 8px;
  margin: 0 5px;
  color: white;
  font-weight: bold;
}

.stat-select {
  margin-top: 10px;
  padding: 5px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.stat-display {
  margin-top: 10px;
}

.stat-label {
  font-size: 16px;
}

.progress-container {
  background: #dcdcdc;
  border-radius: 5px;
  height: 12px;
}

.progress-bar {
  background: #ff69b4;
  height: 100%;
  border-radius: 5px;
}

.random-button, .battle-button, .reset-button {
  margin-top: 10px;
  padding: 12px 22px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
}

.random-button {
  background-color: #32cd32;
  color: white;
}

.battle-button {
  background-color: #ff4500;
  color: white;
}

.reset-button {
  background-color: #4682b4;
  color: white;
}

.disabled-button {
  background-color: #cccccc;
  cursor: not-allowed;
}

.stats-card {
  font-family: 'Courier New', Courier, monospace; 
  color: white; 
  text-shadow: -1px -1px 0 #000, 1px -1px 0 #000, -1px 1px 0 #000, 1px 1px 0 #000; 
  margin: 20px; 
}

</style>
