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
        <h3 class="text-lg font-semibold">{{ pokemon.name }}</h3>
    </div>
</template>