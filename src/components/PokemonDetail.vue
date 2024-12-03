<template>
  <div class="p-4 animate-fade-in">
    <!-- Bouton retour -->
    <button
      @click="goBack"
      class="mb-4 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition"
    >
      ← Retour à la liste
    </button>

    <!-- Détails du Pokémon -->
    <div class="p-4 bg-white shadow rounded max-w-2xl mx-auto">
      <img
        :src="pokemon?.sprites.front_default"
        alt="Image de Pokémon"
        class="w-full h-auto mb-4"
      />
      <h1 class="text-2xl font-bold">{{ pokemon?.name }}</h1>
      <p>
        <strong>Type(s):</strong>
        {{ pokemon?.types.map((type) => type.type.name).join(", ") }}
      </p>
      <p><strong>Taille:</strong> {{ pokemon?.height / 10 }} m</p>
      <p><strong>Poids:</strong> {{ pokemon?.weight / 10 }} kg</p>
      <p><strong>Statistiques:</strong></p>
      <ul>
        <li v-for="stat in pokemon?.stats" :key="stat.stat.name">
          {{ stat.stat.name }}: {{ stat.base_stat }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";

export default {
  name: "PokemonDetail",
  setup() {
    const route = useRoute();
    const router = useRouter();
    const pokemon = ref(null);

    onMounted(async () => {
      const response = await fetch(
        `https://pokeapi.co/api/v2/pokemon/${route.params.id}`
      );
      pokemon.value = await response.json();
    });

    const goBack = () => {
      router.push("/");
    };

    return { pokemon, goBack };
  },
};
</script>
