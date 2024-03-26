<script setup lang="ts">
import { ref } from "vue";
import GameHeader from "./GameHeader.vue";
import BetMenu from "./BetMenu.vue";
import BetRoll from "./BetRoll.vue";
import GameOverScreen from "./GameOverScreen.vue";

const emit = defineEmits(["quitGame"]);
const props = defineProps(["initialCash", "withAudio"]);
const cash = ref(props.initialCash);
const betList = ref([]);
const gameLoopReset = ref(false);
const gameOver = ref(false);
</script>

<template>
  <div class="flex flex-col items-center gap-4">
    <GameHeader
      :cash="cash"
      :initial-cash="props.initialCash"
      @quit-game="emit('quitGame')"
    />
    <BetMenu
      :cash="cash"
      :initial-cash="props.initialCash"
      :gameLoopReset="gameLoopReset"
      @game-loop-reset="gameLoopReset = false"
      @update-cash="
        (newCash) => {
          cash = newCash;
        }
      "
      @send-bet="
        (outputBetList) => {
          betList = outputBetList;
        }
      "
      @game-over="gameOver = true"
    />
    <BetRoll
      v-if="betList.length != 0"
      :betList="betList"
      :with-audio="withAudio"
      @win-amount="(winAmount: Number) => {
        cash += winAmount; betList = []; gameLoopReset = true
      }
     "
    />
    <GameOverScreen v-if="gameOver" @quit-game="emit('quitGame')" />
  </div>
</template>
