<script setup>
const bg = new URL('/src/assets/Nova Mac.jpeg', import.meta.url).href
</script>

<template>
  <div
    class="fixed inset-0 bg-cover bg-center filter blur-md brightness-50 -z-10"
    :style="{ backgroundImage: `url(${bg})` }"
  ></div>
</template>
