<script setup>
import RecipeCard from '../../components/Card.vue'
import axios from 'axios'
import RecipePost from '../../components/RecipePost.vue'
import { ref, onMounted, watch } from 'vue'
import { useUser } from 'vue-clerk'
import LoadingSpinner from '../../components/LoadingSpinner.vue'

const BACKEND_URL = import.meta.env.VITE_APP_BACKEND_URL

const { user } = useUser()
const userId = user.value.id
const userName = user.value.firstName
const userEmail = user.value.primaryEmailAddress

const search = ref('')
const recipes = ref([])
const selectedRecipe = ref({})
const openRecipe = ref(false)
const currentNumberItems = ref(15)
const isLoaded = ref(false)
const canLoadMore = ref(false)
const filteredRecipes = ref([])
const sortCriteria = ref('none')

onMounted(() => {
  fetchData()
})

watch(openRecipe, () => {
  if (!openRecipe.value) {
    fetchData()
  }
})

const fetchData = async () => {
  try {
    axios.get(BACKEND_URL + '/get-recipes').then((response) => {
      recipes.value = response.data.recipes
      if (sortCriteria.value !== 'none') {
        sortRecipes(sortCriteria.value)
      } else {
        filteredRecipes.value = recipes.value
      }
      isLoaded.value = true
    })
  } catch {
    ;(error) => console.log(error)
  }
}

const searchRecipe = () => {
  if (!search.value.trim()) {
    filteredRecipes.value = recipes.value
    return
  }

  try {
    const searchRegex = new RegExp(search.value, 'i')
    filteredRecipes.value = Object.values(recipes.value || {}).filter(
      (recipe) => searchRegex.test(recipe.name) || searchRegex.test(recipe.description || '')
    )
  } catch (error) {
    console.log('Invalid search pattern:', error)
    filteredRecipes.value = recipes.value
  }
}
const setRecipe = (recipe) => {
  selectedRecipe.value = recipe
  openRecipe.value = true
}
const closeModal = () => {
  openRecipe.value = false
}

const sortRecipes = (criteria) => {
  switch (criteria) {
    case 'ingredients':
      filteredRecipes.value = Object.values(filteredRecipes.value).sort(
        (a, b) => (a.ingredients?.length || 0) - (b.ingredients?.length || 0)
      )
      break
    case 'ingredients-desc':
      filteredRecipes.value = Object.values(filteredRecipes.value).sort(
        (a, b) => (b.ingredients?.length || 0) - (a.ingredients?.length || 0)
      )
      break
    default:
      // Reset to original order
      filteredRecipes.value = [...Object.values(recipes.value)]
  }
  sortCriteria.value = criteria
}
</script>

<template>
  <div class="content-wrapper">
    <div class="top">
      <div class="search-container">
        <input
          type="text"
          class="searchbar"
          v-model="search"
          placeholder="Search for recipes here"
          size="50"
          height="20"
          @keydown="searchRecipe"
        />
      </div>
    </div>
    <div class="container-fluid row bottom">
      <div class="try">
        <div class="second justify-content-end">
          <select class="btn" v-model="sortCriteria" @change="sortRecipes(sortCriteria)">
            <option value="none">Sort by (Default)</option>
            <option value="ingredients">Ingredients (Least to Most)</option>
            <option value="ingredients-desc">Ingredients (Most to Least)</option>
          </select>
        </div>
        <div class="container-fluid row results" v-if="isLoaded">
          <RecipeCard
            v-for="recipe in filteredRecipes"
            class="recipecard col-3"
            :key="recipe"
            :title="recipe.name"
            :image="recipe.imageUrl"
            @open-recipe="setRecipe(recipe)"
          />
          <div>
            <button type="button" class="btn load" @click="fetchMore" v-if="canLoadMore">
              Load More
            </button>
          </div>
        </div>
        <LoadingSpinner size="large" v-else />
      </div>
    </div>
    <RecipePost
      v-if="openRecipe"
      :recipe-details="selectedRecipe"
      @close-modal="closeModal"
      :userId="userId"
      :user-name="userName"
      :user-email="userEmail"
    />
  </div>
</template>

<style scoped>
.search-container {
  position: relative;
  display: flex;
  align-items: center;
  max-width: 600px;
  margin: 0 auto;
  padding: 10px;
}
.searchbar {
  width: 100%;
  border: none;
  border-radius: 25px;
  padding: 12px 20px;
  font-size: 16px;
  background-color: white;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.searchbar:focus {
  outline: none;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}
.try {
  height: 100%;
}
.content-wrapper {
  background-color: rgb(255, 254, 245);
  height: 100vh;
  /* padding-inline: 10px; */
  /* border: 1px solid black; */
}
.top {
  background-color: #acbaa1;
  width: 100%;
  align-items: center;
  height: 8vh;
  padding-left: 10px;
}
.bottom {
  height: 90vh;
}
.second {
  display: flex;
  margin-top: 10px;
  height: 6vh;
  justify-content: end;
}
.recipecard {
  margin: 10px 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

.recipecard:hover {
  transform: translateY(-5px);
}
.results {
  overflow: auto;
  height: 90%;
  margin: 10px;
}

.load {
  width: 10%;
  display: block;
  margin: 0 auto;
}
button {
  background-color: #acbaa1;
  color: white;
}

select.btn {
  padding: 8px 12px;
  cursor: pointer;
}

select.btn option {
  background-color: white;
  color: black;
}
</style>
