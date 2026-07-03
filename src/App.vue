<script setup>
  import { ref, onMounted } from 'vue'

  const pokemon = ref([]);
  const loading = ref(false);
  const error = ref('');

  const fetchPokemon = async () => {
    loading.value = true;
    error.value = '';

    try {
      const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=20&offset=0')

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
  <main class="min-h-screen bg-slate-100">
    <div class="flex flex-col items-center">
      <h1 class="text-3xl font-bold text-slate-900">Pokédex</h1>

      <div>
        <h2 class="text-lg font-semibold text-slate-900">Pokemon list</h2>

        <p v-if="loading" class="mt-4 text-slate-600">
          Loading Pokemon...
        </p>

        <p v-else-if="error" class="mt-4 text-red-500">
          {{ error }}
        </p>

        <div v-else>
          <pre>{{pokemon}}</pre>
        </div>

      </div>
    </div>
  </main>
</template>