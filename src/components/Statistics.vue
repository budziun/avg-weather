<template>
  <div
      ref="statisticsRef"
      class="statistics"
      :class="{ 'is-visible': isVisible }"
  >
    <h3>📈 Statystyki pogodowe</h3>

    <div v-if="isVisible" class="stats-grid">
      <StatCard
          emoji="🌡️"
          label="Średnia temperatura"
          :value="averageTemperature"
          unit="°C"
          :style="{ 'animation-delay': '0s' }"
      />

      <StatCard
          emoji="❄️"
          label="Najniższa"
          :value="minTemperature"
          unit="°C"
          :style="{ 'animation-delay': '0.1s' }"
      />

      <StatCard
          emoji="🔥"
          label="Najwyższa"
          :value="maxTemperature"
          unit="°C"
          :style="{ 'animation-delay': '0.2s' }"
      />

      <StatCard
          emoji="✅"
          label="Udane źródła"
          :value="`${successfulSources}/${totalApis}`"
          :style="{ 'animation-delay': '0.3s' }"
      />
    </div>

    <div v-else class="loading-placeholder">
      <div class="placeholder-text">Przygotowywanie statystyk...</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, defineProps } from 'vue'
import StatCard from './StatCard.vue'

const props = defineProps({
  averageTemperature: {
    type: Number,
    required: true
  },
  minTemperature: {
    type: Number,
    required: true
  },
  maxTemperature: {
    type: Number,
    required: true
  },
  successfulSources: {
    type: Number,
    required: true
  },
  totalApis: {
    type: Number,
    required: true
  }
})

const statisticsRef = ref(null)
const isVisible = ref(false)
let observer = null

onMounted(() => {
  // Intersection Observer dla lazy loading
  observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting && !isVisible.value) {
            // Delay dla płynnego renderowania po ApiResults
            setTimeout(() => {
              isVisible.value = true
            }, 300)
          }
        })
      },
      {
        threshold: 0.1,
        rootMargin: '30px'
      }
  )

  if (statisticsRef.value) {
    observer.observe(statisticsRef.value)
  }
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<style scoped>
.statistics {
  background: white;
  padding: 25px;
  border-radius: 15px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.6s ease-out;
}

.statistics.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.statistics h3 {
  color: #2d3436;
  margin-bottom: 20px;
  font-size: 1.4em;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 20px;
}

.loading-placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 120px;
  background: rgba(0, 184, 148, 0.05);
  border-radius: 10px;
  border: 2px dashed rgba(0, 184, 148, 0.3);
}

.placeholder-text {
  color: #00b894;
  font-size: 1.1em;
  font-weight: 500;
  animation: pulse-text 2s ease-in-out infinite;
}

@keyframes pulse-text {
  0%, 100% { opacity: 0.6; }
  50% { opacity: 1; }
}

/* Responsive */
@media (max-width: 768px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .statistics {
    padding: 20px;
  }
}

@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }
}

/* Performance optimization */
.stats-grid {
  will-change: transform;
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .statistics {
    transition: opacity 0.3s ease;
    transform: none;
  }

  .placeholder-text {
    animation: none;
  }
}
</style>
