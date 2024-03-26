<script setup lang="ts"></script>

<template>
  <div
    class="top-0 left-0 absolute w-screen h-screen transition-colors duration-500 flex flex-col justify-center items-center bg-black/80"
  >
    <button class="red-button !bg-transparent">You ran out of cash!</button>
    <button
      class="red-button mt-4"
      @click="$emit('quitGame')"
    >
      Restart 🥲
    </button>
  </div>
</template>
