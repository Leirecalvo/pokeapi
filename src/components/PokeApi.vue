<template>
    <header>
      <img alt="Vue logo" src="../assets/img/logo.svg">
      <div class="pokeFinder">
        <input type="text" v-model="filter" @input="resetResults">
        <ul class="filterResult" v-if="filter !== ''" :class="selectType">
          <li v-for="(pokemon, index) in pokemonFilter" :key="index" @click="selectResult(pokemon.name, $event)"
            :class="[
              pokemon.types[0].type.name.toLowerCase(),
              pokemon.types[1] !== undefined ? pokemon.types[1].type.name.toLowerCase() : '',
            ]">
            {{ pokemon.name }}
          </li>
        </ul>
      </div>
    </header>

    <main>
        <div class="typeFilters">
          <div class="pokeType active" @click="pokemonByType('all'); handleItemClickEvent($event.target)">All</div>
          <div class="pokeType" v-for="(type, index) in pokeTypes" :key="index" @click="pokemonByType(type.name); handleItemClick(index)" :class="{ 'active': activeIndex === index }">
            <img :src="require(`@/assets/img/types/${type.name.toLowerCase()}.svg`)">
            <p>{{ type.name }}</p>
          </div>
        </div>

        <div id="pokemonResult" :class="selectType">
          <div class="pokemon" v-for="(pokemon, index) in pokemonFilter" :key="index" 
            :class="[
              pokemon.types[0].type.name.toLowerCase(),
              pokemon.types[1] !== undefined ? pokemon.types[1].type.name.toLowerCase() : '',
              pokemon.color && pokemon.color.name ? pokemon.color.name : ''
            ]"
          >
            <p class="order">#{{ pokemon.order }}</p>
            <img class="pokeImg" :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/dream-world/${pokemon.id}.svg`" id="">
            <p class="name">{{ pokemon.name }}</p>
            <div class="pokeType">
              <span v-if="pokemon.types[0].type.name !== 'fairy'" >
                <img :src="require(`@/assets/img/types/${pokemon.types[0].type.name.toLowerCase()}.svg`)">
              </span>
              <span v-if="pokemon.types[1] !== undefined && pokemon.types[1].type.name !== 'fairy' && pokemon.types[1].type.name !== 'steel'">
                <img :src="require(`@/assets/img/types/${pokemon.types[1].type.name.toLowerCase()}.svg`)">
              </span>
            </div>
          </div>
        </div>

    </main>
</template>

<script>
import axios from 'axios';
export default {
  name: 'PokeApi',
  props: {
    msg: String
  },
  methods: {
    fetchTypes() {
       axios.get('https://pokeapi.co/api/v2/generation/1')
        .then(response => {
          this.pokeTypes = response.data.types.sort((a, b) => { return a.name.localeCompare(b.name); });
        })
        .catch(error => {
          console.error(error);
        });
    },
    async fetchPokemon() {
      try {
        const response = await axios.get('https://pokeapi.co/api/v2/generation/1');
        this.pokeSpecies = response.data.pokemon_species;

        const requests = [];

        this.pokeSpecies.forEach(pokemon => {
          requests.push(axios.get(pokemon.url), axios.get(`https://pokeapi.co/api/v2/pokemon/${pokemon.name}`));
        });

        const answers = await Promise.all(requests);
        const combinedData = [];
        
        for (let i = 0; i < answers.length; i += 2) {
          const saludoData = answers[i].data;
          const additionalData = answers[i + 1].data;
          const combinedSaludoData = { ...saludoData, ...additionalData };
          combinedData.push(combinedSaludoData);
        }
     
        this.pokemonSpecies = combinedData.sort((a, b) => a.order - b.order);

      } catch (error) {
        console.error(error);
      }
    },
    pokemonByType(type) {
      this.selectType = type;
    },
    removeActive(){
      let activeElement = document.querySelector('.active');
      if (activeElement) {
        activeElement.classList.remove('active');
      }
    },
    handleItemClick(index){
      this.removeActive();
      this.activeIndex = index;
    },
    handleItemClickEvent(clickedElement){
      this.removeActive();
      clickedElement.classList.add('active');
    },
    selectResult(result, event) {
      this.filter = result;
      event.target.parentNode.style.display = 'none';
    },
    resetResults() {
      const ulElement = document.querySelector('.filterResult');
      if (ulElement) {
        ulElement.style.display = '';
      }
    }
  },
  data(){
    return {
      selectType: 'all',
      filter: '',
      pokeTypes: [],
      pokemonSpecies: []
    }
  },
  computed: {
    pokemonFilter() {
      return this.pokemonSpecies.filter(pokemon => pokemon.name.toLowerCase().includes(this.filter.toLowerCase()));
    }
  },
  mounted() {
    this.fetchTypes();
    this.fetchPokemon();
  }
}
</script>
<style src="../assets/styles/poke.css"></style>