<template>
  <v-container class="mt-8">
    <h2 class="text-h4 text-center mb-6">test</h2>
    <v-row justify="center">
      <v-col v-for="cocktail in easyCocktails" :key="cocktail.name" cols="12" md="6" lg="4" class="mb-4">
        <v-card rounded="xl" class="overflow-hidden">
          <v-img :src="cocktail.image" height="250" cover></v-img>
          <v-card-item>
            <v-card-title>{{ cocktail.name }}</v-card-title>
            <v-card-subtitle>{{ cocktail.category }}</v-card-subtitle>
          </v-card-item>
          <v-card-actions class="pa-4">
            <v-btn block rounded="pill" color="orange" @click="toggleRecipe(cocktail.name)">
              {{ isRecipeVisible[cocktail.name] ? 'ซ่อนวิธีทำ' : 'ดูวิธีทำ' }}
            </v-btn>
          </v-card-actions>
          <v-expand-transition>
            <v-card-text v-if="isRecipeVisible[cocktail.name]">
              <p class="font-weight-bold mb-2">ส่วนผสม:</p>
              <ul>
                <li v-for="(ingredient, index) in cocktail.ingredients" :key="index" v-if="Array.isArray(cocktail.ingredients)">
                  {{ ingredient }}
                </li>
              </ul>
              <p class="font-weight-bold mb-2 mt-4">วิธีทำ:</p>
              <p>{{ cocktail.instructions }}</p>
            </v-card-text>
          </v-expand-transition>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const easyCocktailsList = [
  { name: 'Pink Lady' },
  { name: 'Dry Martini' },
  { name: 'Blueberry Mojito' },
];

const easyCocktails = ref([]);
const isRecipeVisible = ref({});

const fetchCocktailDetails = async (cocktailName) => {
  try {
    const encodedName = encodeURIComponent(cocktailName);
    const response = await axios.get(
      `https://www.thecocktaildb.com/api/json/v1/1/search.php?s=${encodedName}`
    );
    if (response.data?.drinks?.[0]) {
      const drink = response.data.drinks[0];
      const ingredientsArray = [];
      for (let i = 1; i <= 15; i++) {
        const ingredient = drink[`strIngredient${i}`];
        const measure = drink[`strMeasure${i}`];
        if (ingredient && ingredient.trim() !== "") {
          ingredientsArray.push(`${measure ? measure.trim() + ' ' : ''}${ingredient.trim()}`);
        }
      }
      return {
        name: drink.strDrink,
        category: drink.strCategory,
        image: drink.strDrinkThumb,
        ingredients: ingredientsArray,
        instructions: drink.strInstructions,
      };
    } else {
      console.warn(`No data found for ${cocktailName}`);
      return { name: cocktailName, category: 'Not Found', image: '', ingredients: [], instructions: 'Recipe not available.' };
    }
  } catch (error) {
    console.error(`Error fetching ${cocktailName}:`, error);
    return { name: cocktailName, category: 'Error', image: '', ingredients: [], instructions: 'Failed to fetch recipe.' };
  }
};

onMounted(async () => {
  for (const cocktailInfo of easyCocktailsList) {
    const details = await fetchCocktailDetails(cocktailInfo.name);
    easyCocktails.value.push(details);
    isRecipeVisible.value[details.name] = false;
  }
});

const toggleRecipe = (cocktailName) => {
  isRecipeVisible.value = {
    ...isRecipeVisible.value,
    [cocktailName]: !isRecipeVisible.value[cocktailName],
  };
};
</script>