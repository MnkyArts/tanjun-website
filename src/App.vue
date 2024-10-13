<script setup>
import Hero from './components/Sections/HeroSection.vue'
import Bento from './components/Elements/BentoGrid.vue'
import Navbar from './components/Elements/AppNavbar.vue'
import AppFooter from './components/Elements/AppFooter.vue'
</script>

<template>
  <header></header>
  <Navbar />
  <main class="min-h-screen">
    <Hero />
    <Bento />
  </main>
  <AppFooter />
</template>

<style scoped></style>
