<script setup>
import Scene from '@/components/Scene.vue';
import Navbar from '@/components/Navbar.vue';
import { ref} from 'vue';

</script>

<template>
  <div class="fixed top-0 left-0 w-full h-screen flex items-center pointer-events-none z-1">
    <Scene ref="scene" />
    <Navbar />
  </div>
  <!-- Scroll sections container -->
  <div class="relative h-[400vh] pointer-events-none">
    <section id="fore-mast" class="h-screen"></section>
    <section id="keel" class="h-screen"></section>
    <section id="bridge" class="h-screen"></section>
    <section id="rudder" class="h-screen"></section>
  </div>
</template>

<style scoped>
</style>
