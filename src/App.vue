<script setup>
import axios from 'axios'
import { ref, computed, watch, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import Loader from './components/Loader.vue';
const baseurl = 'https://api.chucknorris.io/jokes'

const router = useRouter()

const allCategories = ref([]);
const selectedCategory = ref('');
const nameToReplaceWith = ref('');
const loading = ref(false);
const joke = ref({ joke: null, id: '' });
const errorOccurred = ref(false);

async function setCategories() {
  const { data: categories } = await axios.get(`${baseurl}/categories`);
  allCategories.value = [...categories];
}

async function getRandomJoke() {
  if (loading.value) return;
  errorOccurred.value = false;
  loading.value = true;
  try {
    const { data: randomJoke } = await axios.get(`${baseurl}/random${selectedCategory.value !== '' ? `?category=${selectedCategory.value}` : ''}`);
    joke.value.joke = randomJoke.value;
    joke.value.id = randomJoke.id;
  } catch (err) {
    errorOccurred.value = true;
  } finally {
    loading.value = false;
  }
}

async function getJokeById(id) {
  if (loading.value) return;
  errorOccurred.value = false;
  loading.value = true;
  try {
    const { data } = await axios.get(`${baseurl}/${id}`)
    joke.value.joke = data.value;
    joke.value.id = data.id;
  } catch (err) {
    errorOccurred.value = true;
  } finally {
    loading.value = false;
  }
}

watch([nameToReplaceWith, () => joke.value.id], ([name, id]) => {
  const query = {};
  if (name !== '') {
    query.name = name;
  }
  if (id !== '') {
    query.id = id;
  }
  router.replace({ query });
})

const jokeWithReplacedName = computed(() => {
  if (nameToReplaceWith.value === '' || joke.value.joke == null) {
    return joke.value.joke;
  }

  let jokeWithReplacedName = joke.value.joke;
  jokeWithReplacedName = jokeWithReplacedName.replace(/Chuck Norris'|Chuck's/ig, () => {
    if (nameToReplaceWith.value.charAt(nameToReplaceWith.value.length - 1).toLowerCase() === 's') {
      return nameToReplaceWith.value + '\'';
    }
    return nameToReplaceWith.value + '\'s';
  });
  jokeWithReplacedName = jokeWithReplacedName.replace(/Chuck Norris|Chuck/ig, nameToReplaceWith.value);
  return jokeWithReplacedName;
})

onMounted(async () => {
  setCategories();
  await router.isReady();
  const { name, id } = router.currentRoute.value.query;
  if (name) nameToReplaceWith.value = name;
  if (id) {
    joke.value.id = id;
    getJokeById(id);
  }
});
</script>

<template>
  <div class="parent">
    <div>
      <img class="banner-image" src="https://api.chucknorris.io/img/chucknorris_logo_coloured_small@2x.png"
        alt="chuck norris cartoon">
    </div>

    <div class="cartoon-font orange header">Chuck Norris Jokes</div>
    <div class="joke-container">

      <div>
        <div class="replace-name-container">
          <input type="text" placeholder="Chuck Norris" v-model="nameToReplaceWith" />
        </div>

        <div class="joke-buttons-container">
          <div class="category-dropdown-container">
            <div class="select-parent">
              <img src="./assets/triangle-bottom-icon.svg" class="bottom-arrow" alt="">
              <select class="joke-category-select" v-model="selectedCategory">
                <option value="">Random</option>
                <option v-for="category in allCategories" :value="category" :key="category"
                  :selected="category === selectedCategory"> {{ category }}</option>
              </select>
            </div>
          </div>
          <button class="new-joke-button" @click="getRandomJoke">
            New joke!
          </button>
        </div>
      </div>

      <div v-if="loading">
        <Loader />
      </div>
      <div v-else-if="errorOccurred">
        <p class="cartoon-font orange error-message-heading">oops... an error occurred</p>
        <span>better fix your internet connection before Chuck E-roundhouses you. yes, through the screen!</span>
      </div>
      <p v-else>
        {{ jokeWithReplacedName }}
      </p>
    </div>
  </div>
</template>

<style scoped>
.parent {
  padding: 2rem 0;
}

.banner-image {
  display: block;
  width: 100%;
  max-width: 10rem;
  margin: 0 auto;
}

.header {
  font-size: 2rem;
  text-align: center;
}

.joke-container {
  margin: 0 auto;
  max-width: 50rem;
  padding: 0 1rem;
  text-align: center;
}

.joke-buttons-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  margin-top: 1rem;
}

.select-parent {
  position: relative;
}

.bottom-arrow {
  width: 0.5rem;
  height: auto;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  right: 0.75rem;
}

.joke-category-select {
  -webkit-appearance: none;
  -moz-appearance: none;
  border: 2px dotted rgba(255, 94, 0, 0.5);
  padding: 0.5rem 1.5rem 0.5rem 0.5rem;
  font-size: 1rem;
  outline: 0;
  color: white;
}

.category-dropdown-container {
  flex: 0 0 0;
}

.replace-name-container {
  flex: 1 0 0;
}

.replace-name-container>input {
  width: 100%;
  box-sizing: border-box;
  border: 2px dotted rgba(255, 94, 0, 0.5);
  outline: none;
  padding: 0.75rem 1rem;
  margin-top: 1rem;
  font-size: 1rem;
}

.new-joke-button {
  background-color: rgba(255, 94, 0, 0.5);
  border: 2px dotted #242424;
  color: white;
  padding: 0.5rem;
  font-size: 1rem;
}

.error-message-heading {
  font-size: 1.3rem;
  /* text-decoration: underline rgba(255, 94, 0, 0.5) dotted; */
  margin: 0 auto;
  width: fit-content;
  border-bottom: 2px dotted rgba(255, 94, 0, 0.5);
  padding-bottom: 0.25rem;
  margin-bottom: 0.5rem;
}
</style>
