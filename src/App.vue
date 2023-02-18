<script setup lang="ts">

import GuessingGame from "@/components/GuessingGame.vue";
</script>

<template>
  <header>
    <h1>Guess the anime!</h1>
  </header>

  <main>
    <GuessingGame/>
  </main>
</template>

<style lang="scss" scoped>
header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding-block: 1rem;
  width: 100%;
}
main {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100%;
}
</style>
