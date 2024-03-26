<script setup lang="ts">
import { ref } from "vue";

const props = defineProps(["cash", "initialCash"]);
const quitGameCheck = ref(false);
</script>

<template>
  <!-- Header -->
  <div>
    <button
      @click="quitGameCheck = true"
      class="red-button mb-24 xl:mb-auto xl:absolute xl:right-4 xl:top-4"
      v-if="!quitGameCheck"
    >
      Quit Game
    </button>

    <div
      class="mb-24 xl:absolute flex flex-col xl:right-4 xl:top-4 gap-4"
      v-if="quitGameCheck"
    >
      <p class="text-red-500" v-if="quitGameCheck">Are you sure?</p>
      <div class="*:w-full flex gap-2">
        <button @click="quitGameCheck = false" class="secondary-button">
          No
        </button>

        <button
          v-if="quitGameCheck"
          @click="
            () => {
              $emit('quitGame');
              quitGameCheck = false;
            }
          "
          class="red-button"
        >
          Yes
        </button>
      </div>
    </div>

    <p class="xl:absolute xl:left-4 xl:top-4">
      Balance:
      <span
        class="font-xl font-medium text-red-400"
        :class="{ '!text-green-400': props.cash >= props.initialCash }"
        >${{ props.cash }}</span
      >
    </p>
  </div>
</template>

<style>
.red-button {
  background-color: red;
  padding: 6px 10px;
  border-radius: 6px;
  font-weight: 500;
  color: white;
}

.secondary-button {
  background-color: rgb(220, 220, 220);
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid rgb(222, 222, 222);
  font-weight: 500;
  color: black;
}
</style>
