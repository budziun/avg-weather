<template>
  <div class="weather-icon" :class="sizeClass">
    {{ weatherEmoji }}
  </div>
</template>

<script setup>
import { computed, defineProps } from 'vue'

const props = defineProps({
  weatherDescription: {
    type: String,
    default: ''
  },
  size: {
    type: String,
    default: 'large',
    validator: (value) => ['small', 'medium', 'large'].includes(value)
  }
})

const sizeClass = computed(() => `weather-icon--${props.size}`)

const weatherEmoji = computed(() => {
  if (!props.weatherDescription) return '🌤️'

  const description = props.weatherDescription.toLowerCase()

  // Słoneczne warunki
  if (description.includes('bezchmurnie') ||
      description.includes('słonecznie') ||
      description.includes('clear') ||
      description.includes('sunny')) {
    return '☀️'
  }

  // Deszcz
  if (description.includes('deszcz') ||
      description.includes('rain')) {
    return '🌧️'
  }

  // Mżawka
  if (description.includes('mżawka') ||
      description.includes('drizzle')) {
    return '🌦️'
  }

  // Śnieg
  if (description.includes('śnieg') ||
      description.includes('snow')) {
    return '❄️'
  }

  // Burza
  if (description.includes('burza') ||
      description.includes('storm') ||
      description.includes('thunder')) {
    return '⛈️'
  }

  // Mgła
  if (description.includes('mgła') ||
      description.includes('fog') ||
      description.includes('mist')) {
    return '🌫️'
  }

  // Zachmurzenie
  if (description.includes('zachmurzenie') ||
      description.includes('pochmurno') ||
      description.includes('cloud') ||
      description.includes('overcast')) {
    return '☁️'
  }

  // Częściowe zachmurzenie
  if (description.includes('częściowe') ||
      description.includes('partly')) {
    return '⛅'
  }

  // Grad
  if (description.includes('grad') ||
      description.includes('hail')) {
    return '🌨️'
  }

  // Domyślna ikona
  return '🌤️'
})
</script>

<style scoped>
.weather-icon {
  display: inline-block;
  animation: weather-icon-bounce 2s ease-in-out infinite;
  user-select: none;
}

.weather-icon--small {
  font-size: 1.5em;
}

.weather-icon--medium {
  font-size: 2.5em;
}

.weather-icon--large {
  font-size: 4em;
  margin-bottom: 20px;
}

@keyframes weather-icon-bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0) scale(1);
  }
  40% {
    transform: translateY(-5px) scale(1.05);
  }
  60% {
    transform: translateY(-2px) scale(1.02);
  }
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .weather-icon {
    animation: none;
  }
}
</style>
