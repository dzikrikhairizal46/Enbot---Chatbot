<script setup lang="ts">
  import GptChat from './components/GptChat.vue'
</script>

<template>
  <div>
    <nav class="bg-gray-700">
      <div class="container mx-auto px-4">
        <div class="flex items-center justify-between py-4">
          <a class="text-white text-lg font-bold" href="#">
            ChatBot
          </a>
        </div>
      </div>
    </nav>
    <div class="flex justify-center mt-4">
      <div class="w-1/2">
        <GptChat />
      </div>
    </div>
  </div>
</template>
