<template>
  <div
      ref="apiResultsRef"
      class="api-results"
      :class="{ 'is-visible': isVisible }"
  >
    <h3>📊 Szczegółowe dane z poszczególnych źródeł</h3>

    <div v-if="isVisible" class="api-cards">
      <ApiCard
          v-for="(result, index) in apiResults"
          :key="`${result.source}-${index}`"
          :result="result"
          :style="{ 'animation-delay': `${index * 0.1}s` }"
      />
    </div>

    <div v-else class="loading-placeholder">
      <div class="placeholder-text">Ładowanie szczegółowych danych...</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, defineProps } from 'vue'
import ApiCard from './ApiCard.vue'

const props = defineProps({
  apiResults: {
    type: Array,
    required: true,
    default: () => []
  }
})

const apiResultsRef = ref(null)
const isVisible = ref(false)
let observer = null

onMounted(() => {
  // Intersection Observer dla lazy loading
  observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting && !isVisible.value) {
            // Delay dla płynnego renderowania
            setTimeout(() => {
              isVisible.value = true
            }, 200)
          }
        })
      },
      {
        threshold: 0.1,
        rootMargin: '50px'
      }
  )

  if (apiResultsRef.value) {
    observer.observe(apiResultsRef.value)
  }
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<style scoped>
.api-results {
  background: white;
  padding: 25px;
  border-radius: 15px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.6s ease-out;
}

.api-results.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.api-results h3 {
  color: #2d3436;
  margin-bottom: 20px;
  font-size: 1.4em;
}

.api-cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.loading-placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
  background: rgba(116, 185, 255, 0.05);
  border-radius: 10px;
  border: 2px dashed rgba(116, 185, 255, 0.3);
}

.placeholder-text {
  color: #74b9ff;
  font-size: 1.1em;
  font-weight: 500;
  animation: pulse-text 2s ease-in-out infinite;
}

@keyframes pulse-text {
  0%, 100% { opacity: 0.6; }
  50% { opacity: 1; }
}

/* Responsive */
@media (max-width: 1200px) {
  .api-cards {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .api-cards {
    grid-template-columns: 1fr;
  }

  .api-results {
    padding: 20px;
  }
}

/* Performance optimization */
.api-cards {
  will-change: transform;
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .api-results {
    transition: opacity 0.3s ease;
    transform: none;
  }

  .placeholder-text {
    animation: none;
  }
}
</style>
