<template>
  <v-app>
    <v-app-bar density="compact" flat
      style="border-radius: 24px; margin: 16px auto; max-width: 50%; background-color: beige;">
      <v-tabs v-model="activeTab" align-tabs="center" grow slider-color="transparent">

        <v-tab value="home" to="/">
          <v-btn variant="tonal" rounded="pill" :color="activeTab === 'home' ? 'light-green' : 'white'"
            :class="{ 'px-9': activeTab === 'home' }">
            <v-icon start>mdi-home</v-icon>
            <span class="font-weight-medium">Home</span>
          </v-btn>
        </v-tab>

        <v-tab value="cocktailblog" to="/cocktailblog">
          <v-btn variant="tonal" rounded="pill" :color="activeTab === 'cocktailblog' ? 'light-green' : 'white'"
            :class="{ 'px-9': activeTab === 'cocktailblog' }">
            <v-icon start>mdi-liquor</v-icon>
            <span class="font-weight-medium">Cocktail Blog</span>
          </v-btn>
        </v-tab>

        <v-tab value="about" to="/about">
          <v-btn variant="tonal" rounded="pill" :color="activeTab === 'about' ? 'light-green' : 'white'"
            :class="{ 'px-9': activeTab === 'about' }">
            <v-icon start>mdi-account</v-icon>
            <span class="font-weight-medium">About</span>
          </v-btn>
        </v-tab>
      </v-tabs>
    </v-app-bar>
    <!-- main page -->

    <v-main>
      <router-view />
    </v-main>
  </v-app>
  <AppFooter />
</template>

<script setup>
import { ref, watch } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const activeTab = ref(route.name);

watch(() => route.name, (newName) => {
  activeTab.value = newName;
});

</script>

<style scoped>
.v-tab {
  padding: 0 !important;
  /* Remove default padding */
}
</style>
