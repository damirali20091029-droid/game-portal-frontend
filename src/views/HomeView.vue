<script setup>
import { ref, onMounted } from 'vue'
import api from '../api'

const genres = ref([])
const error = ref('')

onMounted(async () => {
  try {
    const response = await api.get('/genres')
    genres.value = response.data.data ?? response.data
  } catch (e) {
    error.value = 'Не удалось загрузить жанры'
  }
})
</script>

<template>
  <h1>Игровой портал</h1>
  <p v-if="error">{{ error }}</p>
  <ul>
    <li v-for="genre in genres" :key="genre.id">{{ genre.name }}</li>
  </ul>
</template>