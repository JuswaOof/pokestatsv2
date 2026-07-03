<script setup>
  import { ref, onMounted } from 'vue';

  import PokemonCard from './components/PokemonCard.vue';

  const pokemon = ref([]);
  const loading = ref(false);
  const error = ref('');

  const fetchPokemon = async () => {
    loading.value = true;
    error.value = '';

    try {
      const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=50&offset=0')

      if (!response.ok) {
        throw new Error('Failed to fetch Pokemon data');
      }

      const data = await response.json();

      pokemon.value = data.results;

      console.log('Pokemon API response:', data);
    } catch (err) {
      error.value = err.message || 'Something went wrong';
    } finally {
      loading.value = false;
    }
  }

  onMounted(() => {
    fetchPokemon();
  })
</script>

<template>
  <main class="min-h-screen">
    <div class="flex flex-col items-center">
      <h1 class="text-3xl font-bold">Pokédex</h1>

      <h2 class="text-lg font-semibold">Pokemon list</h2>
      <div>

        <p v-if="loading" class="mt-4">
          Loading Pokemon...
        </p>

        <p v-else-if="error" class="mt-4 text-red-500">
          {{ error }}
        </p>

        <div v-else class="grid grid-cols-5 gap-4">
            <PokemonCard
              v-for="(item, index) in pokemon"
              :key="index"
              :pokemon="item"
            />
        </div>

      </div>
    </div>
  </main>
</template>