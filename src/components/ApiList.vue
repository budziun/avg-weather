<template>
  <div
      ref="apiListRef"
      class="api-list"
      :class="{ 'is-visible': isVisible }"
  >
    <h3>🔗 Używane API</h3>

    <ul v-if="isVisible">
      <li
          v-for="(api, index) in apiList"
          :key="api.name"
          :style="{ 'animation-delay': `${index * 0.1}s` }"
      >
        <div class="api-icon">{{ api.icon }}</div>
        <div class="api-info">
          <strong>{{ api.name }}</strong>
          <span>{{ api.description }}</span>
        </div>
      </li>
    </ul>

    <div v-else class="loading-placeholder">
      <div class="placeholder-text">Ładowanie informacji o API...</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const apiListRef = ref(null)
const isVisible = ref(false)
let observer = null

const apiList = ref([
  {
    name: 'IMGW-PIB API',
    description: 'Oficjalne dane polskiej służby meteorologicznej',
    icon: '🇵🇱'
  },
  {
    name: 'Open-Meteo API',
    description: 'Europejskie dane meteorologiczne',
    icon: '🌍'
  },
  {
    name: 'Yr.no API',
    description: 'Norweski Instytut Meteorologiczny',
    icon: '🇳🇴'
  },
  {
    name: 'Visual Crossing API',
    description: 'Zaawansowane dane pogodowe',
    icon: '📊'
  },
  {
    name: 'Tomorrow.io API',
    description: 'AI-powered prognozy pogodowe',
    icon: '🤖'
  },
  {
    name: 'AccuWeather API',
    description: 'Globalne dane pogodowe',
    icon: '🌐'
  },
  {
    name: 'Airly API',
    description: 'Polska sieć czujników jakości powietrza',
    icon: '🏭'
  },
  {
    name: '7Timer! API',
    description: 'Specjalistyczne prognozy pogodowe',
    icon: '⭐'
  },
  {
    name: 'WeatherAPI',
    description: 'Darmowy backup API pogodowy',
    icon: '☁️'
  }
])

onMounted(() => {
  // Intersection Observer dla lazy loading
  observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting && !isVisible.value) {
            // Delay dla płynnego renderowania jako ostatni element
            setTimeout(() => {
              isVisible.value = true
            }, 400)
          }
        })
      },
      {
        threshold: 0.1,
        rootMargin: '20px'
      }
  )

  if (apiListRef.value) {
    observer.observe(apiListRef.value)
  }
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<style scoped>
.api-list {
  background: rgba(116, 185, 255, 0.1);
  border: 2px solid rgba(116, 185, 255, 0.3);
  border-radius: 15px;
  padding: 20px;
  margin-top: 25px;
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.6s ease-out;
}

.api-list.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.api-list h3 {
  color: #0984e3;
  margin-bottom: 15px;
  font-size: 1.3em;
}

.api-list ul {
  list-style: none;
  padding-left: 0;
  margin: 0;
}

.api-list li {
  background: white;
  margin: 8px 0;
  padding: 15px;
  border-radius: 8px;
  border-left: 4px solid #74b9ff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease;
  animation: list-item-appear 0.4s ease-out;
  display: flex;
  align-items: center;
  gap: 15px;
}

.api-list li:hover {
  transform: translateX(5px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.api-icon {
  font-size: 1.5em;
  min-width: 30px;
  text-align: center;
}

.api-info {
  flex: 1;
}

.api-info strong {
  display: block;
  color: #2d3436;
  font-size: 1em;
  margin-bottom: 4px;
}

.api-info span {
  color: #636e72;
  font-size: 0.9em;
  line-height: 1.4;
}

.loading-placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100px;
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

@keyframes list-item-appear {
  from { transform: translateX(-15px); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

@keyframes pulse-text {
  0%, 100% { opacity: 0.6; }
  50% { opacity: 1; }
}

/* Performance optimization */
.api-list ul {
  will-change: transform;
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .api-list {
    transition: opacity 0.3s ease;
    transform: none;
  }

  .api-list li {
    animation: none;
  }

  .placeholder-text {
    animation: none;
  }
}
</style>
