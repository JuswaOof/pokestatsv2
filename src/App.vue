<script setup>
  import { ref, onMounted, nextTick, onBeforeUnmount  } from 'vue';
  import PokemonCard from './components/PokemonCard.vue';

  const pokemon = ref([]);
  const loading = ref(false);
  const loadingMore = ref(false);
  const error = ref('');
  const nextPageUrl = ref('https://pokeapi.co/api/v2/pokemon?limit=20&offset=0');

  //infinite scrolling
  const loadMoreTrigger = ref(null)
  let observer = null

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
    if (!nextPageUrl.value || loadingMore.value || loading.value) return

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

  const setupObserver = () => {
      if (!loadMoreTrigger.value) return

      observer = new IntersectionObserver(
        (entries) => {
          const entry = entries[0]

          if (
            entry.isIntersecting &&
            nextPageUrl.value &&
            !loading.value &&
            !loadingMore.value
          ) {
            fetchPokemonPage();
          }
        },
        {
          root: null,
          threshold: 0.1,
        }
      );

      observer.observe(loadMoreTrigger.value);
  }

  onMounted(async () => {
    await fetchPokemonPage();
    await nextTick();
    setupObserver();
  })

  onBeforeUnmount(() => {
    if (observer) {
      observer.disconnect();
    }
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
          <p v-if="loadingMore" class="mt-8 text-center text-slate-500">
            Loading more Pokémon...
          </p>

          <p v-else-if="!nextPageUrl" class="text-slate-500">
            This is the end.
          </p>
        </div>
        <!-- observer -->
        <div ref="loadMoreTrigger" class="h-10"></div>
      </template>
    </div>
  </main>
</template>