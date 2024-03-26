<script setup lang="ts">
import { ref } from "vue";

const cash = ref(50);
</script>

<template>
  <div class="flex flex-col justify-center items-center gap-4">
    <h1 class="font-bold text-center">
      Welcome to
      <span
        class="text-transparent bg-clip-text bg-gradient-to-r from-amber-500 to-pink-500"
        >4D Wonderland</span
      >
    </h1>
    <label class="text-center">Choose your starting cash amount</label>
    <input
      v-model="cash"
      class="w-32 flex-grow-1 text-center"
      type="range"
      min="50"
      max="1000"
      step="50"
    />
    <p class="font-xl font-medium text-green-400">${{ cash }}</p>
    <button @click="$emit('gameStarted', cash)" class="orange-button">
      Start
    </button>
    <div class="flex flex-col items-center gap-2 mt-6">
      <input type="checkbox" id="audio" class="w-6 h-6" @change="$emit('withAudio')"/>
      <label for="audio" class="text-base">Play Audio?</label>
      <p class="text-sm">(Short soundbites when viewing results)</p>
    </div>
  </div>
</template>

<style>
.orange-button {
  background-color: orange;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: large;
  font-weight: 500;
  color: white;
}
</style>
