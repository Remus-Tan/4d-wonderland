<script setup type="ts">
import GameMenu from './components/GameMenu.vue';
import StartMenu from './components/StartMenu.vue';
import { ref } from 'vue';

const gameStarted = ref(false);
const initialCash = ref();
const withAudio = ref(false);

function initializeGame(cash) {
    gameStarted.value = true;
    initialCash.value = cash;
}

function deinitializeGame() {
    gameStarted.value = false;
}

</script>

<template>
  <main
    class="text-2xl h-screen w-screen max-h-screen max-w-screen overflow-hidden flex justify-center items-center"
  >
    <StartMenu
      v-if="!gameStarted"
      @game-started="initializeGame"
      @with-audio="withAudio = !withAudio"
    />
    <GameMenu
      v-if="gameStarted"
      @quit-game="deinitializeGame"
      :initial-cash
      :withAudio
    />
  </main>
</template>
