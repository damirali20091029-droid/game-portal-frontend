<script setup>
import { ref, reactive, onMounted } from 'vue'
import api from '../api'
import GameCard from '../components/GameCard.vue'

const games = ref([])
const genres = ref([])
const meta = ref(null)
const loading = ref(false)
const error = ref('')

const filters = reactive({
  search: '',
  author: '',
  genre_id: '',
  min_rating: '',
  sort: 'rating',
  direction: 'desc',
  page: 1,
})

async function loadGames() {
  loading.value = true
  error.value = ''
  try {
    const params = {}
    for (const [key, value] of Object.entries(filters)) {
      if (value !== '' && value !== null) params[key] = value
    }
    const { data } = await api.get('/games', { params })
    games.value = data.data
    meta.value = data.meta
  } catch (e) {
    error.value =
      e.response?.status === 422 ? 'Проверьте параметры поиска' : 'Не удалось загрузить каталог'
  } finally {
    loading.value = false
  }
}

function applyFilters() {
  filters.page = 1
  loadGames()
}

function resetFilters() {
  Object.assign(filters, {
    search: '', author: '', genre_id: '', min_rating: '',
    sort: 'rating', direction: 'desc', page: 1,
  })
  loadGames()
}

function goToPage(page) {
  filters.page = page
  loadGames()
}

onMounted(async () => {
  try {
    const { data } = await api.get('/genres')
    genres.value = data.data ?? data
  } catch (e) {
    genres.value = []
  }
  loadGames()
})
</script>

<template>
  <h1>Каталог игр</h1>

  <form class="filters" @submit.prevent="applyFilters">
    <input v-model="filters.search" placeholder="Название игры" />
    <input v-model="filters.author" placeholder="Автор" />
    <select v-model="filters.genre_id">
      <option value="">Все жанры</option>
      <option v-for="genre in genres" :key="genre.id" :value="genre.id">{{ genre.name }}</option>
    </select>
    <input v-model="filters.min_rating" type="number" min="0" max="100" placeholder="Мин. рейтинг" />
    <select v-model="filters.sort">
      <option value="rating">По рейтингу</option>
      <option value="created_at">По дате</option>
      <option value="price">По цене</option>
    </select>
    <select v-model="filters.direction">
      <option value="desc">По убыванию</option>
      <option value="asc">По возрастанию</option>
    </select>
    <button type="submit">Найти</button>
    <button type="button" @click="resetFilters">Сбросить</button>
  </form>

  <p v-if="error" class="error">{{ error }}</p>
  <p v-else-if="loading">Загрузка...</p>
  <p v-else-if="!games.length">Ничего не найдено</p>

  <div class="grid">
    <GameCard v-for="game in games" :key="game.id" :game="game" />
  </div>

  <div v-if="meta && meta.last_page > 1" class="pager">
    <button :disabled="meta.current_page <= 1" @click="goToPage(meta.current_page - 1)">Назад</button>
    <span>Страница {{ meta.current_page }} из {{ meta.last_page }}</span>
    <button :disabled="meta.current_page >= meta.last_page" @click="goToPage(meta.current_page + 1)">Вперёд</button>
  </div>
</template>