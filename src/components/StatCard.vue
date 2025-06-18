<template>
  <div class="stat-item">
    <div class="stat-emoji">{{ emoji }}</div>
    <strong>{{ label }}</strong><br>
    {{ displayValue }}
  </div>
</template>

<script setup>
import { computed, defineProps } from 'vue'

const props = defineProps({
  emoji: {
    type: String,
    required: true
  },
  label: {
    type: String,
    required: true
  },
  value: {
    type: [String, Number],
    required: true
  },
  unit: {
    type: String,
    default: ''
  }
})

const displayValue = computed(() => {
  if (props.value === null || props.value === undefined) {
    return 'Brak danych'
  }
  return `${props.value}${props.unit}`
})
</script>

<style scoped>
.stat-item {
  text-align: center;
  padding: 20px;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-radius: 12px;
  border: 2px solid #dee2e6;
  transition: all 0.2s ease;
  animation: stat-appear 0.6s ease-out;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  background: linear-gradient(135deg, #e9ecef, #f8f9fa);
}

.stat-emoji {
  font-size: 2em;
  margin-bottom: 10px;
  animation: bounce 1.5s ease-in-out infinite;
}

.stat-item strong {
  color: #2d3436;
  font-size: 0.9em;
  display: block;
  margin-bottom: 5px;
}

@keyframes stat-appear {
  from { transform: scale(0.95) translateY(8px); opacity: 0; }
  to { transform: scale(1) translateY(0); opacity: 1; }
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-3px); }
  60% { transform: translateY(-1px); }
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .stat-item {
    animation: none;
  }

  .stat-emoji {
    animation: none;
  }
}
</style>
