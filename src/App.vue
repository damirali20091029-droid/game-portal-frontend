<script setup>
import { RouterLink, RouterView, useRouter } from 'vue-router'
import { useAuthStore } from './stores/auth'

const auth = useAuthStore()
const router = useRouter()

async function logout() {
  await auth.logout()
  router.push('/login')
}
</script>

<template>
  <header>
    <nav>
      <RouterLink to="/">Каталог</RouterLink>
      <template v-if="auth.isLoggedIn">
        <RouterLink to="/games/new">Опубликовать игру</RouterLink>
        <RouterLink class="user-link" :to="`/users/${auth.user?.id}`">
          {{ auth.user?.name }}
        </RouterLink>
        <button @click="logout">Выйти</button>
      </template>
      <template v-else>
        <RouterLink class="user-link" to="/login">Вход</RouterLink>
        <RouterLink to="/register">Регистрация</RouterLink>
      </template>
    </nav>
  </header>
  <main>
    <RouterView />
  </main>
</template>