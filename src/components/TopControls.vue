<template>
  <div class="top-controls">
    <!-- 當前時間顯示 -->
    <div class="current-time">
      {{ currentTime }}
    </div>
    
    <!-- 主題切換按鈕 -->
    <button @click="$emit('toggle-theme')" class="theme-toggle-btn" :title="isDarkTheme ? '切換至淺色主題' : '切換至深色主題'">
      <span class="theme-icon">{{ isDarkTheme ? '☀️' : '🌙' }}</span>
      <span class="theme-text">{{ isDarkTheme ? '淺色' : '深色' }}</span>
    </button>
  </div>
</template>

<script>
export default {
  name: 'TopControls',
  props: {
    currentTime: {
      type: String,
      required: true
    },
    isDarkTheme: {
      type: Boolean,
      required: true
    }
  },
  emits: ['toggle-theme']
}
</script>

<style scoped>
/* 頂部控制列 */
.top-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  gap: 1rem;
}

.current-time {
  background-color: rgba(0, 0, 0, 0.8);
  padding: 1rem;
  border-radius: 8px;
  color: #61dafb;
  font-size: 1.2rem;
  font-weight: 500;
}

/* 主題切換按鈕 */
.theme-toggle-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1.25rem;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  position: relative;
  overflow: hidden;
}

.theme-toggle-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.3);
  background: linear-gradient(135deg, #764ba2, #667eea);
}

.theme-toggle-btn:active {
  transform: translateY(0);
}

.theme-icon {
  font-size: 1.2rem;
  transition: transform 0.3s ease;
}

.theme-toggle-btn:hover .theme-icon {
  transform: rotate(180deg);
}

.theme-text {
  font-size: 0.85rem;
  letter-spacing: 0.025em;
}

/* 主題切換時的樣式 */
body.light-theme .current-time {
  background-color: rgba(255, 255, 255, 0.9);
  color: #213547;
  border: 1px solid #ddd;
}

/* 響應式設計 */
@media (max-width: 768px) {
  .top-controls {
    flex-direction: column;
    gap: 1rem;
    margin-bottom: 1.5rem;
  }

  .theme-toggle-btn {
    order: -1;
    align-self: flex-end;
  }

  .theme-text {
    display: none;
  }
}
</style>
