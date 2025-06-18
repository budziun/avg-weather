<template>
  <div class="search-box">
    <input
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        @keyup.enter="handleSearch"
        :placeholder="placeholder"
        :disabled="loading"
        class="search-input"
    >
    <button
        @click="handleSearch"
        :disabled="loading || !modelValue.trim()"
        class="search-button"
    >
      <span v-if="loading" class="loading-spinner"></span>
      <span v-else>{{ loading ? 'Ładowanie...' : 'Sprawdź pogodę' }}</span>
    </button>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  loading: {
    type: Boolean,
    default: false
  },
  placeholder: {
    type: String,
    default: 'Wpisz nazwę miasta w Polsce (np. Olsztyn, Warszawa, Kraków)...'
  }
})

const emit = defineEmits(['update:modelValue', 'search'])

const handleSearch = () => {
  if (!props.loading && props.modelValue.trim()) {
    emit('search', props.modelValue.trim())
  }
}
</script>

<style scoped>
.search-box {
  display: flex;
  gap: 15px;
  margin-bottom: 30px;
  align-items: center;
  animation: slide-in-up 0.5s ease-out;
}

.search-input {
  flex: 1;
  padding: 15px 20px;
  border: 2px solid #ddd;
  border-radius: 25px;
  font-size: 1.1em;
  outline: none;
  transition: all 0.2s ease;
  font-family: 'Poppins', sans-serif;
}

.search-input:focus {
  border-color: #74b9ff;
  box-shadow: 0 0 0 3px rgba(116, 185, 255, 0.2);
  transform: scale(1.01);
}

.search-input:disabled {
  background-color: #f8f9fa;
  cursor: not-allowed;
}

.search-button {
  padding: 15px 30px;
  background: linear-gradient(135deg, #74b9ff, #0984e3);
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 1.1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 4px 15px rgba(116, 185, 255, 0.4);
  font-family: 'Poppins', sans-serif;
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 150px;
  justify-content: center;
}

.search-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(116, 185, 255, 0.6);
}

.search-button:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.loading-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top: 2px solid white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes slide-in-up {
  from { transform: translateY(15px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Responsive */
@media (max-width: 768px) {
  .search-box {
    flex-direction: column;
  }

  .search-input,
  .search-button {
    width: 100%;
  }
}
</style>
