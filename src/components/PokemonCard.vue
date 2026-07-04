<script setup>
    defineProps({
        pokemon: {
            type: Object,
            required: true
        }
    });
</script>

<template>
    <div class="shadow p-4 m-2 cursor-pointer">
        <p class="mt-4 text-sm text-slate-500">
            #{{ pokemon.id.toString().padStart(3, '0') }}
        </p>
        <h3 class="text-lg font-semibold">{{ pokemon.name }}</h3>
        <img
            :src="pokemon.image"
            :alt="pokemon.name"
            class="mx-auto h-28 w-28 object-contain"
        />
        <div class="mt-3 flex flex-wrap gap-2">
            <span
                v-for="type in pokemon.types"
                :key="type"
                class="rounded-full bg-slate-100 px-3 py-1 text-sm capitalize text-slate-700"
            >
                {{ type }}
            </span>
        </div>
    </div>
</template>