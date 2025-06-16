<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
import data from '@/data/data.json';

const points = data.points;
const activePoint = ref(0);
const progress = ref(0);

const handleScroll = () => {
  const scrollPosition = window.scrollY;
  const windowHeight = window.innerHeight;
  const documentHeight = document.documentElement.scrollHeight;
  
  // Calculate scroll percentage
  const scrollPercentage = (scrollPosition / (documentHeight - windowHeight)) * 100;
  progress.value = Math.min(scrollPercentage, 100);
  
  // Calculate which section is active
  const sectionHeight = windowHeight;
  const currentSection = Math.floor(scrollPosition / sectionHeight);
  activePoint.value = Math.min(currentSection, points.length - 1);
  
//   console.log('Scroll:', { scrollPosition, progress: progress.value, activePoint: activePoint.value });
};

const scrollToSection = (index) => {
  const targetPosition = index * window.innerHeight;
  window.scrollTo({
    top: targetPosition,
    behavior: 'smooth'
  });
};

onMounted(() => {
  window.addEventListener('scroll', handleScroll);
  // Initial calculation
  handleScroll();
});

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll);

  window.addEventListener('resize', () => {
    $screen.md = window.matchMedia('(min-width: 768px)').matches
  })
});

const $screen = {
  md: window.matchMedia('(min-width: 768px)').matches
}
</script>

<template>
  <nav class="fixed md:right-28 lg:right-48 rounded z-50 top-8 left-1/2 -translate-x-1/2 w-[80%] h-1 md:left-auto md:top-1/2 md:-translate-x-0 md:-translate-y-1/2 md:h-[80%] md:w-auto flex items-center">
    <div class="relative w-full h-1 grid grid-cols-4 gap-0 md:h-full md:w-1 md:grid-cols-none md:grid-rows-4">
      <!-- Background line -->
      <div class="absolute bg-[#601a11] opacity-30 w-full h-1 md:h-full md:w-1"></div>
      
      <!-- Progress bar -->
      <div 
        class="absolute bg-[#601a11] rounded origin-left md:origin-top transition-all duration-300 ease-out h-1 md:w-1"
        :style="[
          $screen.md ? { height: `${progress}%` } : { width: `${progress}%` }
        ]"
      ></div>
      
      <!-- Navigation points -->
      <div 
      v-for="(point, i) in points" 
        :key="i" 
        :id="point.label.toLowerCase().replace(/\s+/g, '-')"
        @click="scrollToSection(i)"
        class="absolute w-4 h-4 bg-white border-2 border-[#601a11] rounded-full shadow-[0_0_8px_2px_rgba(96,26,17,0.4)] cursor-pointer z-10 transition-all duration-300 hover:bg-[#601a11] hover:scale-150 hover:shadow-[0_0_12px_3px_rgba(96,26,17,0.5)] -top-1.5 md:-left-1.5 md:top-auto"
        :style="[
          $screen.md ? { top: point.position } : { left: point.position }
        ]"
        :class="{
          '!bg-[#601a11] scale-110 md:scale-125': progress >= point.threshold
        }"
      >
        <span class="absolute whitespace-nowrap text-[#601a11] top-6 left-1/2 -translate-x-1/2 md:left-6 md:top-[-8px] md:translate-x-0">
          {{ point.label }}
        </span>
      </div>

              <!-- Info Card -->
              <div
         v-for="(point, i) in points"
         :key="i"
         >
         <div
          v-if="point.description && progress >= point.threshold && progress < (points[i + 1]?.threshold || 101)"
          class="absolute bg-white rounded-lg p-4 shadow-lg border border-[#601a11] w-64 transition-opacity duration-300"
          :class="$screen.md ? 
            '-left-[17rem] top-1/2 -translate-y-1/2' : 
            'fixed left-1/2 top-16 -translate-x-1/2 z-50'"
        >
          <h3 class="font-bold text-[#601a11] mb-2">{{ point.label }}</h3>
          <p class="text-sm text-gray-700">{{ point.description }}</p>
        </div>
         </div>
    </div>
  </nav>
</template>

<style scoped>
#end-point {
    display: none;
}
</style>