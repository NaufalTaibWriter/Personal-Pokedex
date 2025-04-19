<script setup>
import { ref, onMounted } from 'vue';
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090/');
const records = ref([]); // Reactive object for the record data
const pokemonInput = ref(''); // Input field for Pokémon name

const fightingPokemonApi = "https://pokeapi.co/api/v2/type/2"; // Fighting Pokémon API
const fightingPokemon = ref([]); // Reactive array for Pokémon data


function capturePokemon() {
  if (pokemonInput.value.trim() === '') {
    alert('Please enter a Pokémon name.');
    return;
  }

  const pokemonName = {
    item: pokemonInput.value,
    captured: true
  }
  pb.collection('pokemonCollection').create(pokemonName)
    .then((record) => {
      console.log('Pokémon captured:', record);
      records.value.push(record); // Add the new record to the list
      pokemonInput.value = ''; // Clear the input field
    })
    .catch((error) => {
      console.error('Error capturing Pokémon:', error);
      alert('Failed to capture Pokémon. Please try again.');
    });
}

function getFightingPokemon() {
  fetch(fightingPokemonApi)
    .then((response) => response.json())
    .then((data) => {
      if (Array.isArray(data.pokemon)) {
        fightingPokemon.value = data.pokemon;
      } else {
        console.error("Unexpected API response:", data);
        fightingPokemon.value = [];
      }
    })
    .catch((error) => {
      console.error("Error fetching Pokémon:", error);
      fightingPokemon.value = [];
    });
}

function getPokemonId(url) {
  const parts = url.split('/');
  return parts[parts.length - 2];
}

function toTitleCase(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

onMounted(() => {
  pb.collection('pokemonCollection').getFullList().then(data => records.value = data);
});
</script>

<template>
  <div class="container mx-auto p-4">
    <!-- Fighting Pokémon Section -->
    <div class="bg-gradient-to-r from-yellow-400 to-red-500 rounded-lg shadow-lg p-6 mb-8">
      <h1 class="text-3xl font-bold text-white text-center mb-4">Fighting Pokémon</h1>
      <button 
        class="bg-white hover:bg-gray-100 text-gray-800 font-semibold py-2 px-4 border border-gray-400 rounded shadow mx-auto block"
        @click="getFightingPokemon">
        Get Fighting Pokémon
      </button>
      <p class="text-white text-center mt-4">Displayed Pokémon: {{ fightingPokemon.slice(0, 73).length }}</p>
      <div v-if="fightingPokemon.length > 0" class="grid grid-cols-3 gap-6 mt-6">
        <div 
          v-for="pokemonData in fightingPokemon.slice(0, 73)" 
          :key="pokemonData.pokemon.url" 
          class="bg-white rounded-lg shadow-md p-4 text-center hover:scale-105 transform transition">
          <img 
            :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${getPokemonId(pokemonData.pokemon.url)}.png`" 
            :alt="pokemonData.pokemon.name" 
            class="w-20 h-20 mx-auto mb-2" />
          <p class="text-lg font-semibold text-gray-800">{{ toTitleCase(pokemonData.pokemon.name) }}</p>
        </div>
      </div>
      <p v-else class="text-white text-center mt-4">Loading Pokémon or no Pokémon found.</p>
    </div>

    <!-- Pokémon Collection Section -->
    <div class="bg-gradient-to-r from-blue-400 to-green-500 rounded-lg shadow-lg p-6">
      <h1 class="text-3xl font-bold text-white text-center mb-4">Pokémon Collection</h1>
      <input type="text" 
        v-model="pokemonInput" 
        placeholder="Enter Pokémon name" 
        class="border border-gray-300 rounded-lg p-2 mb-4 w-full" />
      <ul class="space-y-2">
        <li 
          v-for="record in records" 
          :key="record.id" 
          class="bg-white rounded-lg shadow-md p-4 flex items-center justify-between">
          <span class="text-gray-800 font-semibold">{{ record.id }} - {{ toTitleCase(record.pokemon) }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.container {
  font-family: 'Arial', sans-serif;
}

button {
  transition: all 0.3s ease;
}

button:hover {
  transform: scale(1.05);
}

li {
  list-style-type: none;
}

.grid div:hover {
  background-color: #f9fafb;
}
</style>