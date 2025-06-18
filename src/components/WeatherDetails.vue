<template>
  <div class="weather-details">
    <div class="detail-item">
      <div class="detail-emoji">💧</div>
      <strong>Wilgotność</strong><br>
      {{ humidity }}%
    </div>
    <div class="detail-item">
      <div class="detail-emoji">🌡️</div>
      <strong>Ciśnienie</strong><br>
      {{ pressure }} hPa
    </div>
    <div class="detail-item">
      <div class="detail-emoji">💨</div>
      <strong>Wiatr</strong><br>
      {{ windSpeed }} m/s
    </div>
  </div>
</template>

<script setup>
import { defineProps } from 'vue'

const props = defineProps({
  humidity: {
    type: [String, Number],
    default: 'Brak danych'
  },
  pressure: {
    type: [String, Number],
    default: 'Brak danych'
  },
  windSpeed: {
    type: [String, Number],
    default: 'Brak danych'
  }
})
</script>

<style scoped>
.weather-details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 20px;
  margin-top: 25px;
}

.detail-item {
  background: rgba(255, 255, 255, 0.2);
  padding: 15px;
  border-radius: 10px;
  text-align: center;
  backdrop-filter: blur(10px);
  transition: all 0.2s ease;
  animation: detail-appear 0.5s ease-out;
}

.detail-item:hover {
  transform: translateY(-3px);
  background: rgba(255, 255, 255, 0.3);
}

.detail-emoji {
  font-size: 1.5em;
  margin-bottom: 8px;
  animation: bounce 1.5s ease-in-out infinite;
}

@keyframes detail-appear {
  from { transform: translateY(10px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-3px); }
  60% { transform: translateY(-1px); }
}

/* Responsive */
@media (max-width: 768px) {
  .weather-details {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .weather-details {
    grid-template-columns: 1fr;
  }
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .detail-item {
    animation: none;
  }

  .detail-emoji {
    animation: none;
  }
}
</style>
