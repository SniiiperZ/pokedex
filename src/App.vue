<template>
  <div>
    <!-- En-tête -->
    <header class="bg-blue-500 text-white p-4 text-center text-xl font-bold">
      <a href="/" class="hover:underline">Pokédex</a>
    </header>

    <!-- Vue routeur -->
    <div class="container mx-auto p-4">
      <router-view />
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
};
</script>
