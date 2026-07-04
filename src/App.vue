<script setup>
  import { ref, onMounted } from 'vue';
  import PokemonCard from './components/PokemonCard.vue';

  const pokemon = ref([]);
  const loading = ref(false);
  const loadingMore = ref(false);
  const error = ref('');
  const nextPageUrl = ref('https://pokeapi.co/api/v2/pokemon?limit=20&offset=0');

  const fetchPokemonDetails = async (pokemonList) => {
    const detailPromises = pokemonList.map(async (item) => {
      const response = await fetch(item.url);

      if (!response.ok) {
        throw new Error(`Failed to fetch details for ${item.name}`);
      }

      const data = await response.json();

      return {
        id: data.id,
        name: data.name,
        image:
          data.sprites.other.home.front_default
          ? data.sprites.other.home.front_default
          : data.sprites.other.dream_world.front_default
          ? data.sprites.other.dream_world.front_default
          : data.sprites.front_default,
        types: data.types.map((typeInfo) => typeInfo.type.name),
      }
    });

    return Promise.all(detailPromises);
  }

  const fetchPokemonPage = async () => {
    if (!nextPageUrl.value) return

    const isInitialLoad = pokemon.value.length === 0

    if (isInitialLoad) {
      loading.value = true
    } else {
      loadingMore.value = true
    }

    error.value = ''

    try {
      const response = await fetch(nextPageUrl.value);

      if (!response.ok) {
        throw new Error('Failed to fetch Pokemon data');
      }

      const data = await response.json();
      const detailedPokemon = await fetchPokemonDetails(data.results);

      pokemon.value = [...pokemon.value, ...detailedPokemon];
      nextPageUrl.value = data.next; // Update the next page URL for pagination
    } catch (err) {
      error.value = err.message || 'Something went wrong';
    } finally {
      loading.value = false;
      loadingMore.value = false;
    }
  }

  onMounted(() => {
    fetchPokemonPage();
  })
</script>

<template>
  <main class="min-h-screen">
    <div class="flex flex-col items-center">
      <h1 class="text-3xl font-bold">Pokédex</h1>
      <h2 class="text-lg font-semibold">Pokemon list</h2>

      <p v-if="loading" class="mt-4">
        Loading Pokemon...
      </p>

      <p v-else-if="error" class="mt-4 text-red-500">
        {{ error }}
      </p>

      <template v-else>
        <div class="grid grid-cols-5 gap-4">
          <PokemonCard
            v-for="item in pokemon"
            :key="item.id"
            :pokemon="item"
          />
        </div>

        <div class="my-8 flex justify-center">
          <button
            v-if="nextPageUrl"
            @click="fetchPokemonPage"
            :disabled="loadingMore"
            class="rounded-lg bg-blue-900 px-4 py-2 text-white disabled:cursor-not-allowed cursor-pointer disabled:opacity-50"
          >
            {{ loadingMore ? 'Loading more...' : 'Load more Pokemon' }}
          </button>

          <p v-else class="text-slate-500">
            This is the end.
          </p>
        </div>
      </template>
    </div>
  </main>
</template>