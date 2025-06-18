<template>
  <div class="api-card" :class="cardClass">
    <div class="api-header">
      <strong>{{ result.source }}</strong>
      <span class="status-badge" :class="badgeClass">
        {{ result.success ? '✅' : '❌' }}
      </span>
    </div>

    <div class="api-content">
      <div v-if="result.success" class="api-success-data">
        <div class="temp-display">{{ result.temperature }}°C</div>
        <div class="api-details">
          <div v-if="result.humidity" class="api-detail-item">
            <strong>💧 Wilgotność:</strong> {{ result.humidity }}%
          </div>
          <div v-if="result.pressure" class="api-detail-item">
            <strong>🌡️ Ciśnienie:</strong> {{ result.pressure }} hPa
          </div>
          <div v-if="result.windSpeed" class="api-detail-item">
            <strong>💨 Wiatr:</strong> {{ result.windSpeed }} m/s
          </div>
          <div v-if="result.description" class="api-detail-item">
            <strong>📝 Opis:</strong> {{ result.description }}
          </div>
        </div>
      </div>

      <div v-else class="api-error">
        <div class="error-icon">⚠️</div>
        <div class="error-text">{{ result.error }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, defineProps } from 'vue'

const props = defineProps({
  result: {
    type: Object,
    required: true,
    validator: (value) => {
      return value && typeof value === 'object' &&
          'success' in value && 'source' in value
    }
  }
})

const cardClass = computed(() => {
  return props.result.success ? 'success' : 'error'
})

const badgeClass = computed(() => {
  return props.result.success ? 'success' : 'error'
})
</script>

<style scoped>
.api-card {
  border: 2px solid #ddd;
  border-radius: 12px;
  padding: 20px;
  transition: all 0.2s ease;
  animation: card-appear 0.5s ease-out;
  background: white;
}

.api-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
}

.api-card.success {
  border-color: #00b894;
  background: rgba(0, 184, 148, 0.05);
}

.api-card.error {
  border-color: #e74c3c;
  background: rgba(231, 76, 60, 0.05);
}

.api-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 1px solid #eee;
}

.api-header strong {
  color: #2d3436;
  font-size: 1.1em;
}

.status-badge {
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 0.8em;
  font-weight: bold;
  animation: badge-pulse 1.5s ease-in-out infinite;
}

.status-badge.success {
  background: rgba(0, 184, 148, 0.2);
  color: #00a085;
}

.status-badge.error {
  background: rgba(231, 76, 60, 0.2);
  color: #c0392b;
}

.api-success-data {
  text-align: center;
}

.temp-display {
  font-size: 2em;
  font-weight: bold;
  color: #0984e3;
  margin-bottom: 15px;
  animation: temp-glow 2s ease-in-out infinite;
}

.api-details {
  display: grid;
  gap: 8px;
  text-align: left;
}

.api-detail-item {
  font-size: 0.9em;
  color: #636e72;
  padding: 5px 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.api-detail-item:last-child {
  border-bottom: none;
}

.api-error {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 15px;
  background: rgba(231, 76, 60, 0.1);
  border-radius: 8px;
  animation: error-wiggle 0.3s ease-in-out;
}

.error-icon {
  font-size: 1.2em;
}

.error-text {
  color: #e74c3c;
  font-style: italic;
  flex: 1;
}

/* Animacje */
@keyframes card-appear {
  from { transform: translateY(15px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes badge-pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.8; }
}

@keyframes temp-glow {
  0%, 100% { text-shadow: 0 0 8px rgba(9, 132, 227, 0.3); }
  50% { text-shadow: 0 0 15px rgba(9, 132, 227, 0.5); }
}

@keyframes error-wiggle {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-0.5deg); }
  75% { transform: rotate(0.5deg); }
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .api-card,
  .status-badge,
  .temp-display,
  .api-error {
    animation: none;
  }
}
</style>
