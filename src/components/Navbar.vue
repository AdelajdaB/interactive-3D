<script setup>
import { onMounted, onUnmounted, ref } from 'vue';

const points = [
  { label: 'Fore Mast', id: 'fore-mast', threshold: 20, position: '20%' },
  { label: 'Keel', id: 'keel', threshold: 40, position: '40%' },
  { label: 'Bridge', id: 'bridge', threshold: 60, position: '60%' },
  { label: 'Rudder', id: 'rudder', threshold: 80, position: '80%' },
  { label: 'end-point', id: 'end', threshold: 100, position: '100%' }
];

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
});
</script>

<template>
  <nav class="fixed right-28 lg:right-48 top-1/2 transform -translate-y-1/2 h-[80%] flex items-center z-50">
    <div class="relative h-full w-1 grid grid-rows-4 gap-0">
      <!-- Background line -->
      <div class="absolute h-full w-1 bg-[#601a11] opacity-30"></div>
      
      <!-- Progress bar -->
      <div 
        class="absolute w-1 bg-[#601a11] origin-top transition-all duration-300 ease-out"
        :style="{ height: `${progress}%` }"
      ></div>
      
      <!-- Navigation points -->
      <div 
        v-for="(point, i) in points" 
        :key="i" 
        :id="point.label.toLowerCase().replace(/\s+/g, '-')"
        @click="scrollToSection(i)"
        class="absolute w-4 h-4 bg-white -left-1.5 border-2 border-[#601a11] rounded-full cursor-pointer z-10 transition-all duration-300 hover:bg-[#601a11] hover:scale-150"
        :style="{ top: point.position }"
        :class="{
          '!bg-[#601a11] scale-120': progress >= point.threshold
        }"
      >
        <span class="absolute left-6 top-[-8px] whitespace-nowrap text-[#601a11]">
          {{ point.label }}
        </span>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.scale-120 {
  transform: scale(1.2);
}

#end-point {
    display: none;
}
</style>