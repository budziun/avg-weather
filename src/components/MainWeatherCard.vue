<template>
  <div class="main-weather">
    <WeatherIcon
        :weather-description="weatherDescription"
        size="large"
    />
    <h2>{{ cityName }}</h2>
    <div class="temperature">{{ temperature }}°C</div>
    <div class="weather-description-large">{{ weatherDescription }}</div>
    <div class="temperature-note">
      Średnia z {{ successfulSources }} źródeł
      <span v-if="temperatureRange">({{ temperatureRange }})</span>
    </div>
    <p>{{ description }}</p>
  </div>
</template>

<script setup>
import { defineProps } from 'vue'
import WeatherIcon from './WeatherIcon.vue'

const props = defineProps({
  cityName: {
    type: String,
    required: true
  },
  temperature: {
    type: Number,
    required: true
  },
  weatherDescription: {
    type: String,
    required: true
  },
  temperatureRange: {
    type: String,
    default: ''
  },
  successfulSources: {
    type: Number,
    required: true
  },
  description: {
    type: String,
    default: ''
  }
})
</script>

<style scoped>
.main-weather {
  background: linear-gradient(135deg, #74b9ff, #0984e3);
  color: white;
  padding: 30px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 15px 30px rgba(116, 185, 255, 0.4);
  position: relative;
  overflow: hidden;
  /* USUNIĘTE ANIMACJE DLA OPTYMALIZACJI */
}

.main-weather h2 {
  font-size: 2em;
  margin-bottom: 15px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  position: relative;
  z-index: 1;
}

.temperature {
  font-size: 4em;
  font-weight: bold;
  margin: 20px 0;
  text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.2);
  position: relative;
  z-index: 1;
  /* USUNIĘTA ANIMACJA temperature-pulse */
}

.weather-description-large {
  font-size: 1.8em;
  font-weight: 600;
  margin: 20px 0;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.2);
  position: relative;
  z-index: 1;
  /* USUNIĘTA ANIMACJA text-glow */
}

.temperature-note {
  font-size: 1.1em;
  margin-bottom: 15px;
  opacity: 0.9;
  position: relative;
  z-index: 1;
}

/* Responsive */
@media (max-width: 768px) {
  .main-weather h2 {
    font-size: 1.8em;
  }

  .temperature {
    font-size: 3em;
  }

  .weather-description-large {
    font-size: 1.4em;
  }
}
</style>
