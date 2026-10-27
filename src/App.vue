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
        <span>{{ auth.user?.name }}</span>
        <button @click="logout">Выйти</button>
      </template>
      <template v-else>
        <RouterLink to="/login">Вход</RouterLink>
        <RouterLink to="/register">Регистрация</RouterLink>
      </template>
    </nav>
  </header>
  <main>
    <RouterView />
  </main>
</template>