<template>
  <div>
    <input class="search" v-model="search" placeholder="Search for Pokemon" />
    <button class="btn" @click="fetchPokemon">Search</button>
  </div>

  <div class="list">
    <p v-if="!pokemons && !error">Use the field to search Pokemon</p>

    <p v-if="!pokemons && error">{{ error }}</p>

    <div
      @click="statusPokemon(pokemon.name, pokemon.img)"
      class="card"
      v-else
      v-for="pokemon of pokemons"
      :key="pokemon.name"
    >
      <img :src="pokemon.img" :alt="pokemon.name" />
      <p class="capitalize">
        {{ pokemon.name }}
      </p>
    </div>
  </div>
  <div v-if="open" class="card modal">
    <img :src="modalUrl" :alt="modelName" />

    <p class="capitalize">{{ modelName }}</p>
    <li>Base experience: {{ modalPokemon.base_experience }}</li>
    <div class="list">
      <p for="hp">
        HP:
        <progress id="hp" :value="modalPokemon.stats[0].base_stat" max="255" />
        {{ modalPokemon.stats[0].base_stat }}
      </p>
      <p>
        Attack:
        <progress :value="modalPokemon.stats[1].base_stat" max="255" />
        {{ modalPokemon.stats[1].base_stat }}
      </p>
      <p>
        Defense:
        <progress :value="modalPokemon.stats[2].base_stat" max="255" />
        {{ modalPokemon.stats[2].base_stat }}
      </p>
      <p>
        Special-attack:
        <progress :value="modalPokemon.stats[3].base_stat" max="255" />
        {{ modalPokemon.stats[3].base_stat }}
      </p>
      <p>
        Special-defense:
        <progress :value="modalPokemon.stats[4].base_stat" max="255" />
        {{ modalPokemon.stats[4].base_stat }}
      </p>
      <p>
        Speed: <progress :value="modalPokemon.stats[5].base_stat" max="255" />
        {{ modalPokemon.stats[5].base_stat }}
      </p>
    </div>
    <details>
      <summary>Moves</summary>
      <div class="listMoves">
        <p class="capitalize movesStyles" v-for="move of moves" :key="move">
          {{ move }}
        </p>
      </div>
    </details>
    <button class="btn" @click="open = false">Close</button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "HomeView",
  data() {
    return {
      pokemons: null,
      moves: null,
      search: "",
      img: null,
      loading: false,
      open: false,
      modelName: "",
      modalUrl: "",
      modalPokemon: null,
      error: null,
    };
  },
  methods: {
    async fetchPokemon() {
      this.loading = true;

      const species = await axios
        .get(
          `https://pokeapi.co/api/v2/pokemon-species/${this.search.toLowerCase()}`
        )
        .then()
        .catch((e) => (this.error = e.response.data));

      console.log(species);
      if (species !== "Not Found") {
        const evolution = await axios.get(species.data.evolution_chain.url);

        let evolveTo = "";

        const pokeNames = evolution.data.chain.evolves_to.map((evolve) => {
          evolveTo = evolve.evolves_to;
          return `https://pokeapi.co/api/v2/pokemon/${evolve.species.name}`;
        });

        evolveTo.map((x) => {
          pokeNames.push(`https://pokeapi.co/api/v2/pokemon/${x.species.name}`);
        });

        pokeNames.unshift(
          `https://pokeapi.co/api/v2/pokemon/${evolution.data.chain.species.name}`
        );

        const list = axios.all(pokeNames.map((url) => axios.get(url))).then(
          axios.spread((...res) => {
            const data = res.map((x) => {
              return { name: x.data.name, img: x.data.sprites.front_default };
            });
            return data;
          })
        );

        this.pokemons = await list;

        this.loading = false;
      }
    },
    async statusPokemon(name, url) {
      this.open = true;
      this.modelName = name;
      this.modalUrl = url;
      const getstatus = await axios.get(
        `https://pokeapi.co/api/v2/pokemon/${name}`
      );
      this.modalPokemon = getstatus.data;
    },
  },
};
</script>
<style>
.card {
  background-color: white;
  border-radius: 5px;
  padding: 10px;

  margin: 10px;
}
.capitalize {
  text-transform: capitalize;
}
.list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: center;
  padding: 10px;
}
.listMoves {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: center;
  margin: 10px;
}
.movesStyles {
  box-shadow: 3px 5px lightgray;
  border-radius: 5px;
  padding: 10px;
  margin: 5px;
}

.btn {
  padding: 10px;
  background-color: blue;
  border: none;
  border-radius: 5px;
  color: white;
  box-shadow: 3px 5px lightgray;
}

.search {
  padding: 9px;
  border: none;
  border-radius: 5px;
  margin-right: 5px;
}

.modal {
  position: fixed;
  z-index: 999;
  padding: 10px;
  height: 500px;
  overflow: auto;
  top: 5%;
  left: 10%;
  right: 10%;
  box-shadow: 2px 5px lightgray;
}
</style>
