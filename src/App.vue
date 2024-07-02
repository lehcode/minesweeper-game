<script setup lang="ts">
import GameBoard from '@/components/GameBoard.vue'
</script>

<template>
  <div class="flex flex-col items-center justify-center">
    <header>
      <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
    </header>

    <main>
      <GameBoard />
    </main>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
  justify-content: center;
}

.logo {
  display: block;
  margin: 0 0 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
  }

  .logo {
    margin: 0;
  }
}
</style>

