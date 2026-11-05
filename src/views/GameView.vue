<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter, RouterLink } from 'vue-router'
import api from '../api'
import { useAuthStore } from '../stores/auth'

const route = useRoute()
const router = useRouter()
const auth = useAuthStore()

const game = ref(null)
const loading = ref(true)
const notFound = ref(false)
const error = ref('')

const comments = computed(() => game.value?.comments?.data ?? game.value?.comments ?? [])
const canManage = computed(
  () => auth.isLoggedIn && (auth.isAdmin || auth.user?.id === game.value?.author?.id),
)

function formatDate(value) {
  return new Date(value).toLocaleDateString('ru-RU')
}

async function loadGame() {
  try {
    const { data } = await api.get(`/games/${route.params.id}`)
    game.value = data.data ?? data
  } catch (e) {
    if (e.response?.status === 404) notFound.value = true
    else error.value = 'Не удалось загрузить игру'
  } finally {
    loading.value = false
  }
}

async function removeGame() {
  if (!confirm('Удалить игру? Это действие нельзя отменить.')) return
  try {
    await api.delete(`/games/${game.value.id}`)
    router.push('/')
  } catch (e) {
    error.value = 'Не удалось удалить игру'
  }
}

onMounted(loadGame)
</script>

<template>
  <p v-if="loading">Загрузка...</p>
  <p v-else-if="notFound">Игра не найдена. <RouterLink to="/">Вернуться в каталог</RouterLink></p>
  <p v-else-if="error && !game" class="error">{{ error }}</p>

  <article v-else-if="game" class="game">
    <h1>{{ game.title }}</h1>
    <p class="meta">
      Автор: {{ game.author?.name }} · Добавлена {{ formatDate(game.created_at) }}
    </p>

    <div class="badges">
      <span class="badge">Рейтинг {{ game.rating }}</span>
      <span class="badge price">
        {{ Number(game.price) === 0 ? 'Бесплатно' : game.price + ' ₽' }}
      </span>
      <span v-for="genre in game.genres" :key="genre.id" class="badge">{{ genre.name }}</span>
    </div>

    <div v-if="game.screenshots?.length" class="gallery">
      <img v-for="shot in game.screenshots" :key="shot.id" :src="shot.url" :alt="game.title" />
    </div>

    <p class="description">{{ game.description }}</p>

    <div v-if="canManage" class="actions">
      <button type="button" @click="removeGame">Удалить игру</button>
    </div>
    <p v-if="error" class="error">{{ error }}</p>

    <section class="comments">
      <h2>Комментарии ({{ game.comments_count ?? comments.length }})</h2>
      <p v-if="!comments.length">Пока нет комментариев</p>
      <div v-for="comment in comments" :key="comment.id" class="comment">
        <strong>{{ comment.user?.name }}</strong>
        <span class="meta"> · {{ formatDate(comment.created_at) }}</span>
        <p>{{ comment.body }}</p>
      </div>
    </section>
  </article>
</template>