<template>
  <div v-if="isVisible" class="loading-container">
    <div class="loading-content">
      <h3>🔄 Pobieranie danych z {{ totalApis }} źródeł...</h3>

      <div class="progress-container">
        <div class="progress-bar" :style="{ width: progressPercentage + '%' }"></div>
        <div class="progress-glow" :style="{ width: progressPercentage + '%' }"></div>
      </div>

      <div class="api-status-grid">
        <div
            v-for="(status, api) in apiStatus"
            :key="api"
            class="status-card"
            :class="getStatusClass(status)"
        >
          <div class="status-icon"></div>
          <div class="status-content">
            <strong>{{ api }}</strong>
            <span>{{ status }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, defineProps } from 'vue'

const props = defineProps({
  totalApis: {
    type: Number,
    required: true
  },
  completedApis: {
    type: Number,
    required: true
  },
  apiStatus: {
    type: Object,
    default: () => ({})
  },
  isVisible: {
    type: Boolean,
    default: true
  }
})

const progressPercentage = computed(() => {
  return Math.round((props.completedApis / props.totalApis) * 100)
})

const getStatusClass = (status) => {
  if (status.includes('Gotowe') || status.includes('✅')) return 'success'
  if (status.includes('Błąd') || status.includes('❌')) return 'error'
  return 'loading'
}
</script>

<style scoped>
.loading-container {
  text-align: center;
  padding: 30px;
  background: rgba(116, 185, 255, 0.1);
  border-radius: 15px;
  margin-bottom: 20px;
  animation: pulse-glow 1.5s ease-in-out infinite;
}

.loading-content h3 {
  margin-bottom: 20px;
  color: #2d3436;
  font-size: 1.2em;
}

.progress-container {
  position: relative;
  width: 100%;
  height: 8px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  margin: 20px 0;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #74b9ff, #0984e3);
  border-radius: 4px;
  transition: width 0.3s ease;
  position: relative;
}

.progress-bar::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 30px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.4), transparent);
  animation: progress-shine 1.5s ease-in-out infinite;
}

.progress-glow {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.4), transparent);
  border-radius: 4px;
  animation: progress-shine 2s ease-in-out infinite;
}

.api-status-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 10px;
  margin-top: 20px;
}

.status-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  padding: 10px 15px;
  display: flex;
  align-items: center;
  gap: 12px;
  transition: all 0.2s ease;
  animation: status-appear 0.4s ease-out;
}

.status-card.success {
  border-left: 4px solid #00b894;
  background: rgba(0, 184, 148, 0.1);
  color: #00a085;
  animation: success-pulse 1.5s ease-in-out infinite;
}

.status-card.error {
  border-left: 4px solid #e74c3c;
  background: rgba(231, 76, 60, 0.1);
  color: #c0392b;
  animation: error-shake 0.3s ease-in-out;
}

.status-card.loading {
  border-left: 4px solid #74b9ff;
  background: rgba(116, 185, 255, 0.1);
  color: #0984e3;
  animation: loading-pulse 1s ease-in-out infinite;
}

.status-icon {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  animation: pulse 2s ease-in-out infinite;
}

.status-card.success .status-icon {
  background: #00d4aa;
}

.status-card.error .status-icon {
  background: #ff6b6b;
}

.status-card.loading .status-icon {
  background: #667eea;
}

.status-content {
  flex: 1;
  text-align: left;
}

.status-content strong {
  display: block;
  font-size: 0.9em;
  margin-bottom: 4px;
}

.status-content span {
  font-size: 0.8em;
  opacity: 0.8;
}

/* Animacje */
@keyframes pulse-glow {
  0%, 100% { box-shadow: 0 0 15px rgba(116, 185, 255, 0.3); }
  50% { box-shadow: 0 0 25px rgba(116, 185, 255, 0.5); }
}

@keyframes progress-shine {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(200%); }
}

@keyframes status-appear {
  from { transform: scale(0.9); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

@keyframes success-pulse {
  0%, 100% { border-left-color: #00b894; }
  50% { border-left-color: #00d4aa; }
}

@keyframes error-shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-2px); }
  75% { transform: translateX(2px); }
}

@keyframes loading-pulse {
  0%, 100% { border-left-color: #74b9ff; }
  50% { border-left-color: #0984e3; }
}

@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.8; transform: scale(1.05); }
}

/* Responsive */
@media (max-width: 768px) {
  .api-status-grid {
    grid-template-columns: 1fr;
  }
}
</style>
