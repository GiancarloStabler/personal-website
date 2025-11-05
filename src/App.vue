<script setup>
import Navbar2 from './components/Navbar2.vue'
const bg = new URL('./assets/Nova Mac.png', import.meta.url).href
</script>

<template>
  <div class="relative min-h-screen text-white overflow-hidden">
    <!-- Background -->
    <div
      class="fixed inset-0 bg-cover bg-center filter blur-md brightness-80 -z-10"
      :style="{ backgroundImage: `url(${bg})` }"
    ></div>

    <!-- Navbar -->
    <Navbar2 />

    <!-- Content -->
    
  </div>
</template>
