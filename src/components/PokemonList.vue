<template>
  <div class="animate-fade-in">
    <h1 class="text-2xl font-bold mb-4">Pokédex</h1>

    <!-- Barre de recherche -->
    <input
      v-model="searchQuery"
      placeholder="Cherche un Pokémon par nom ou numéro..."
      class="p-2 border rounded w-full mb-4"
    />

    <!-- Menu déroulant pour les types -->
    <select v-model="selectedType" class="p-2 border rounded w-full mb-4">
      <option value="">Tous les types</option>
      <option v-for="type in types" :key="type.name" :value="type.name">
        {{ type.name }}
      </option>
    </select>

    <!-- Liste des Pokémon -->
    <div class="grid grid-cols-5 sm:grid-cols-6 lg:grid-cols-8 gap-4">
      <div
        v-for="pokemon in paginatedPokemons"
        :key="pokemon.id"
        class="p-2 bg-white shadow rounded cursor-pointer transform hover:scale-105 transition duration-200 ease-in-out"
        @click="openModal(pokemon)"
      >
        <!-- Image et nom -->
        <div class="relative w-full h-24">
          <img
            :src="pokemon.image"
            :alt="pokemon.name"
            loading="lazy"
            class="w-full h-full object-contain"
          />
        </div>
        <h3 class="text-center mt-1 text-sm font-medium truncate">
          {{ pokemon.name }}
        </h3>
      </div>
    </div>

    <!-- Modal -->
    <div
      v-if="selectedPokemon"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
      @click.self="closeModal"
    >
      <div
        class="bg-white p-6 rounded-lg shadow-lg w-full max-w-md sm:max-w-lg lg:max-w-xl relative"
      >
        <!-- Bouton pour fermer le modal -->
        <button
          @click="closeModal"
          class="absolute top-2 right-2 text-gray-600 hover:text-gray-800 text-2xl"
        >
          &times;
        </button>

        <!-- Contenu du modal -->
        <div class="text-center">
          <img
            :src="selectedPokemon.sprites.front_default"
            alt="Image de Pokémon"
            class="w-32 h-32 mx-auto"
          />
          <h2 class="text-2xl font-bold mt-4">{{ selectedPokemon.name }}</h2>
          <p class="mt-2">
            <strong>Type(s):</strong>
            {{ selectedPokemon.types.map((t) => t.type.name).join(", ") }}
          </p>
          <p><strong>Taille:</strong> {{ selectedPokemon.height / 10 }} m</p>
          <p><strong>Poids:</strong> {{ selectedPokemon.weight / 10 }} kg</p>
          <div class="mt-4">
            <strong>Statistiques:</strong>
            <ul class="mt-2 space-y-1">
              <li
                v-for="stat in selectedPokemon.stats"
                :key="stat.stat.name"
                class="text-sm"
              >
                {{ stat.stat.name }}: {{ stat.base_stat }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div class="flex justify-center items-center mt-6 space-x-4">
      <button
        @click="loadPreviousPage"
        :disabled="currentPage === 1"
        class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition disabled:opacity-50"
      >
        Précédent
      </button>
      <span class="text-lg">Page {{ currentPage }}</span>
      <button
        @click="loadNextPage"
        :disabled="!hasMorePokemons"
        class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition disabled:opacity-50"
      >
        Suivant
      </button>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from "vue";

export default {
  name: "PokemonList",
  setup() {
    const allPokemons = ref([]);
    const pokemons = ref([]);
    const types = ref([]);
    const searchQuery = ref("");
    const selectedType = ref("");
    const currentPage = ref(1);
    const limit = 32;
    const hasMorePokemons = ref(true);
    const errorMessage = ref("");
    const selectedPokemon = ref(null); // Pokémon sélectionné pour le modal
    const cache = {}; // Cache des requêtes

    const fetchPokemonDetails = async (url) => {
      if (cache[url]) {
        return cache[url];
      }
      const response = await fetch(url);
      if (!response.ok) throw new Error("Erreur de chargement des données.");
      const data = await response.json();
      cache[url] = data;
      return data;
    };

    const fetchAllPokemons = async () => {
      try {
        const response = await fetch(
          "https://pokeapi.co/api/v2/pokemon?limit=10000&offset=0"
        );
        if (!response.ok)
          throw new Error(
            "Impossible de charger la liste complète des Pokémon."
          );
        const data = await response.json();

        allPokemons.value = await Promise.all(
          data.results.map(async (pokemon) => {
            const details = await fetchPokemonDetails(pokemon.url);
            return {
              id: details.id,
              name: pokemon.name,
              image: details.sprites.front_default,
              types: details.types.map((t) => t.type.name),
            };
          })
        );
      } catch (error) {
        errorMessage.value = error.message;
      }
    };

    const fetchTypes = async () => {
      try {
        const response = await fetch("https://pokeapi.co/api/v2/type/");
        if (!response.ok)
          throw new Error("Impossible de charger les types de Pokémon.");
        const data = await response.json();
        types.value = data.results.filter(
          (t) => t.name !== "unknown" && t.name !== "shadow"
        );
      } catch (error) {
        errorMessage.value = error.message;
      }
    };

    const openModal = async (pokemon) => {
      try {
        selectedPokemon.value = await fetchPokemonDetails(
          `https://pokeapi.co/api/v2/pokemon/${pokemon.id}`
        );
      } catch (error) {
        errorMessage.value = error.message;
      }
    };

    const closeModal = () => {
      selectedPokemon.value = null;
    };

    const filteredPokemons = computed(() => {
      return allPokemons.value.filter((pokemon) => {
        const matchesSearch =
          pokemon.name
            .toLowerCase()
            .includes(searchQuery.value.toLowerCase()) ||
          pokemon.id.toString().includes(searchQuery.value);
        const matchesType =
          !selectedType.value || pokemon.types.includes(selectedType.value);

        return matchesSearch && matchesType;
      });
    });

    const paginatedPokemons = computed(() => {
      const start = (currentPage.value - 1) * limit;
      const end = start + limit;
      return filteredPokemons.value.slice(start, end);
    });

    const loadNextPage = () => {
      currentPage.value++;
    };

    const loadPreviousPage = () => {
      if (currentPage.value > 1) {
        currentPage.value--;
      }
    };

    onMounted(() => {
      fetchAllPokemons();
      fetchTypes();
    });

    return {
      allPokemons,
      paginatedPokemons,
      filteredPokemons,
      fetchPokemonDetails,
      errorMessage,
      selectedPokemon,
      fetchTypes,
      searchQuery,
      selectedType,
      currentPage,
      hasMorePokemons,
      loadNextPage,
      loadPreviousPage,
      openModal,
      closeModal,
      types,
    };
  },
};
</script>
