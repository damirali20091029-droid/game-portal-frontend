<script setup>
import { RouterLink } from 'vue-router'

defineProps({ game: { type: Object, required: true } })
</script>

<template>
  <article class="card">
    <img v-if="game.screenshots?.length" :src="game.screenshots[0].url" :alt="game.title" />
    <div v-else class="placeholder"></div>
    <h3>
      <RouterLink :to="`/games/${game.id}`">{{ game.title }}</RouterLink>
    </h3>
    <p>Автор: {{ game.author?.name }}</p>
    <p>{{ game.genres?.map((g) => g.name).join(', ') }}</p>
    <div class="badges">
      <span class="badge">Рейтинг {{ game.rating }}</span>
      <span class="badge price">{{ Number(game.price) === 0 ? 'Бесплатно' : game.price + ' ₽' }}</span>
    </div>
  </article>
</template>