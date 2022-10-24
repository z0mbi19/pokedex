<template>
  <div>
    <input v-model="search" placeholder="Search for Pokemon" />
    <button @click="fetchPokemon">Search</button>
  </div>
  <div v-if="pokemon" class="card">
    <img :src="pokemon.sprites.front_default" :alt="pokemon.name" />
    <p>{{ pokemon.name }}</p>
    <div>
      <li>Base experience: {{ pokemon.base_experience }}</li>
      <li>
        HP: <progress :value="pokemon.stats[0].base_stat" max="255" />
        {{ pokemon.stats[0].base_stat }}
      </li>
      <li>
        Attack: <progress :value="pokemon.stats[1].base_stat" max="255" />
        {{ pokemon.stats[1].base_stat }}
      </li>
      <li>
        Defense: <progress :value="pokemon.stats[2].base_stat" max="255" />
        {{ pokemon.stats[2].base_stat }}
      </li>
      <li>
        Special-attack:
        <progress :value="pokemon.stats[3].base_stat" max="255" />
        {{ pokemon.stats[3].base_stat }}
      </li>
      <li>
        Special-defense:
        <progress :value="pokemon.stats[4].base_stat" max="255" />
        {{ pokemon.stats[4].base_stat }}
      </li>
      <li>
        Speed: <progress :value="pokemon.stats[5].base_stat" max="255" />
        {{ pokemon.stats[5].base_stat }}
      </li>
      <details>
        <summary>Moves</summary>
        <li v-for="move of moves" :key="move">{{ move }}</li>
      </details>
    </div>
  </div>
  <div class="list">
    <p v-if="!pokemons">Use the field to search Pokemon</p>

    <div
      @click="statusPokemon(pokemon.name, pokemon.img)"
      class="card"
      v-else
      v-for="pokemon of pokemons"
      :key="pokemon.name"
    >
      <img :src="pokemon.img" :alt="pokemon.name" />
      <p>
        {{ pokemon.name }}
      </p>
    </div>
  </div>
  <div v-if="open" class="card modal">
    <img :src="modalUrl" :alt="modelName" />

    <p>{{ modelName }}</p>
    <button @click="open = false">Close</button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "HomeView",
  data() {
    return {
      pokemons: null,
      pokemon: null,
      moves: null,
      search: "",
      img: null,
      loading: false,
      open: false,
      modelName: "",
      modalUrl: "",
    };
  },
  methods: {
    async fetchPokemon() {
      this.loading = true;
      const pokemonSearch = await axios.get(
        `https://pokeapi.co/api/v2/pokemon/${this.search}`
      );
      this.pokemon = pokemonSearch.data;
      const getMoves = pokemonSearch.data.moves.map((move) => {
        return move.move.name;
      });
      this.moves = getMoves;
      const species = await axios.get(
        `https://pokeapi.co/api/v2/pokemon-species/${this.search}`
      );
      const evolution = await axios.get(species.data.evolution_chain.url);
      const pokeNames = evolution.data.chain.evolves_to.map((evolve) => {
        return `https://pokeapi.co/api/v2/pokemon/${evolve.species.name}`;
      });

      const list = axios.all(pokeNames.map((url) => axios.get(url))).then(
        axios.spread((...res) => {
          const data = res.map((x) => {
            return { name: x.data.name, img: x.data.sprites.front_default };
          });
          return data;
        })
      );

      console.log(list);

      this.pokemons = await list;

      this.loading = false;
    },
    async statusPokemon(name, url) {
      this.open = true;
      this.modelName = name;
      this.modalUrl = url;
      const getstatus = await axios.get(
        `https://pokeapi.co/api/v2/pokemon/${name}`
      );
      console.log(getstatus.data);
    },
  },
};
</script>
<style>
.card {
  background-color: white;
  border-radius: 5px;
  margin: 10px;
}
.list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: center;
}
.modal {
  position: fixed;
  z-index: 999;
  top: 20%;
  left: 50%;
  width: 300px;
  margin-left: -150px;
}
</style>
