<template>
  <div 
    class="timer-card"
    :class="getTimerStatusClass(timer)"
  >
    <div class="timer-left">
      <div class="boss-info">
        <img 
          :src="getBossImage(timer.boss.name)" 
          :alt="timer.boss.name"
          class="boss-image"
          @error="onImageError"
        />
        <div class="boss-text">
          <div class="boss-name">{{ timer.boss.name }}</div>
          <div class="channel-name">CH.{{ timer.channelName }}</div>
        </div>
      </div>
      <div class="location">
        <div v-if="Array.isArray(timer.boss.location)">
          <div v-for="(loc, index) in timer.boss.location" :key="index" class="location-item">
            📍 {{ loc }}
          </div>
        </div>
        <div v-else class="location-item">
          📍 {{ timer.boss.location }}
        </div>
      </div>
    </div>
    
    <div class="timer-center">
      <div class="timer-status-badge">
        {{ getStatusText(timer) }}
      </div>
      <div 
        class="timer-display"
        :class="getTimerDisplayClass(timer)"
      >
        {{ formatTime(timer.timeRemaining, timer) }}
      </div>
      <div class="timer-info">
        <div>開始：{{ formatDateTime(timer.startTime) }}</div>
        <div>重生：{{ formatDateTime(timer.minEndTime) }} ~ {{ formatDateTime(timer.maxEndTime) }}</div>
      </div>
    </div>

    <div class="timer-right">
      <button @click="$emit('share-timer', timer.id)" class="share-btn btn-modern">
        <span class="btn-icon">📤</span>
        <span class="btn-text">分享</span>
      </button>
      <button @click="$emit('reset-timer', timer.id)" class="reset-btn btn-modern">
        <span class="btn-icon">🔄</span>
        <span class="btn-text">重設</span>
      </button>
      <button @click="$emit('remove-timer', timer.id)" class="remove-btn btn-modern">
        <span class="btn-icon">🗑️</span>
        <span class="btn-text">移除</span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TimerCard',
  props: {
    timer: {
      type: Object,
      required: true
    }
  },
  emits: ['reset-timer', 'remove-timer', 'share-timer'],
  methods: {
    // 獲取計時器狀態
    getTimerStatus(timer) {
      const now = new Date()
      const elapsedMinutes = Math.floor((now - timer.startTime) / (1000 * 60))
      
      if (elapsedMinutes < timer.boss.minRespawnMinutes) {
        return 'waiting' // 等待中
      } else if (elapsedMinutes <= timer.boss.maxRespawnMinutes) {
        return 'ready' // 可能重生
      } else {
        return 'overdue' // 超時
      }
    },

    // 獲取狀態文字
    getStatusText(timer) {
      const status = this.getTimerStatus(timer)
      switch (status) {
        case 'waiting':
          return '⏳ 等待重生'
        case 'ready':
          return '✨ 可能重生'
        case 'overdue':
          return '🔥 已超時'
        default:
          return '❓ 未知狀態'
      }
    },

    // 獲取計時器卡片樣式類別
    getTimerStatusClass(timer) {
      const status = this.getTimerStatus(timer)
      return `timer-${status}`
    },

    // 獲取計時器顯示樣式類別
    getTimerDisplayClass(timer) {
      const status = this.getTimerStatus(timer)
      return {
        'timer-waiting': status === 'waiting',
        'timer-ready': status === 'ready',
        'timer-overdue': status === 'overdue'
      }
    },

    // 格式化時間顯示 (支援小時:分鐘:秒)
    formatTime(seconds, timer) {
      const status = this.getTimerStatus(timer)
      
      if (status === 'overdue') {
        const now = new Date()
        const overtimeMinutes = Math.floor((now - timer.maxEndTime) / (1000 * 60))
        return `已超時 ${this.formatDuration(overtimeMinutes * 60)}`
      }
      
      if (seconds <= 0) {
        return "可能重生！"
      }
      
      return this.formatDuration(seconds)
    },

    // 格式化時間長度 (自動顯示小時、分鐘、秒)
    formatDuration(totalSeconds) {
      const hours = Math.floor(totalSeconds / 3600)
      const minutes = Math.floor((totalSeconds % 3600) / 60)
      const seconds = totalSeconds % 60

      if (hours > 0) {
        // 顯示小時:分鐘:秒
        return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
      } else {
        // 只顯示分鐘:秒
        return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
      }
    },

    // 格式化日期時間
    formatDateTime(date) {
      return date.toLocaleString('zh-TW', {
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit'
      })
    },

    // 獲取 BOSS 圖片路徑
    getBossImage(bossName) {
      try {
        // 使用 Vite 的動態導入來獲取圖片
        return new URL(`../assets/images/${bossName}.png`, import.meta.url).href
      } catch (error) {
        console.warn(`無法載入 BOSS 圖片: ${bossName}`, error)
        return new URL('../assets/images/default.png', import.meta.url).href
      }
    },

    // 圖片載入錯誤處理
    onImageError(event) {
      // 如果圖片載入失敗，隱藏圖片元素
      event.target.style.display = 'none'
      console.warn('BOSS 圖片載入失敗:', event.target.src)
    }
  }
}
</script>

<style scoped>
.timer-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.5rem;
  gap: 2rem;
  border: 1px solid #333;
  border-radius: 12px;
  background-color: #1a1a1a;
  min-height: 100px;
  transition: background-color 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
  /* 防止尺寸變化 */
  box-sizing: border-box;
  width: 100%;
}

/* 計時器狀態樣式 */
.timer-waiting {
  border: 4px solid #61dafb;
  background-color: #1a1a1a;
  transition: background-color 0.3s ease, border-color 0.3s ease;
}

.timer-ready {
  border: 4px solid #4CAF50;
  background-color: #1a2e1a;
  box-shadow: 0 0 10px rgba(76, 175, 80, 0.3);
  transition: background-color 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
}

.timer-overdue {
  border: 4px solid #ff6b6b;
  background-color: #2e1a1a;
  box-shadow: 0 0 10px rgba(255, 107, 107, 0.3);
  transition: background-color 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
}

/* 狀態徽章基本樣式 */
.timer-status-badge {
  font-size: 0.9rem;
  font-weight: bold;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  margin-bottom: 0.5rem;
  display: inline-block;
  transition: all 0.3s ease;
}

/* 深色主題狀態徽章 */
.timer-card.timer-waiting .timer-status-badge {
  background-color: rgba(97, 218, 251, 0.2);
  color: #61dafb;
  border: 1px solid #61dafb;
  transition: all 0.3s ease;
}

.timer-card.timer-ready .timer-status-badge {
  background-color: rgba(76, 175, 80, 0.2);
  color: #4CAF50;
  border: 1px solid #4CAF50;
  transition: all 0.3s ease;
}

.timer-card.timer-overdue .timer-status-badge {
  background-color: rgba(255, 107, 107, 0.2);
  color: #ff6b6b;
  border: 1px solid #ff6b6b;
  transition: all 0.3s ease;
}

.timer-left {
  flex: 1;
  text-align: left;
}

.boss-info {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 0.75rem;
}

.boss-image {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
  border: 2px solid #333;
  background-color: #2a2a2a;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.boss-image:hover {
  transform: scale(1.05);
}

.boss-text {
  flex: 1;
}

.boss-name {
  font-size: 1.25rem;
  color: #ffa500;
  margin-bottom: 0.25rem;
  font-weight: bold;
}

.channel-name {
  font-size: 1rem;
  color: #888;
}

.timer-center {
  flex: 2;
  text-align: center;
}

.timer-right {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: center;
}

.timer-display {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  transition: color 0.3s ease;
  /* 防止字體尺寸影響佈局 */
  line-height: 1.1;
  min-height: 2.75rem;
    padding: 1rem 0;
}

/* 深色主題下的計時器顯示顏色 */
.timer-display.timer-waiting {
  color: #61dafb;
}

.timer-display.timer-ready {
  color: #4CAF50;
  text-shadow: 0 0 10px rgba(76, 175, 80, 0.5);
  animation: pulse-ready 2s infinite;
}

.timer-display.timer-overdue {
  color: #ff6b6b;
  text-shadow: 0 0 10px rgba(255, 107, 107, 0.5);
  animation: pulse-overdue 1s infinite;
}

@keyframes pulse-ready {
  0%, 100% { 
    opacity: 1; 
    transform: scale(1);
  }
  50% { 
    opacity: 0.8; 
    transform: scale(1.02);
  }
}

@keyframes pulse-overdue {
  0%, 100% { 
    opacity: 1; 
    transform: scale(1);
  }
  50% { 
    opacity: 0.7; 
    transform: scale(1.05);
  }
}

.location {
  color: #888;
  font-size: 0.9rem;
}

.location-item {
  margin-bottom: 0.2rem;
  line-height: 1.3;
}

.location-item:last-child {
  margin-bottom: 0;
}

.timer-info {
  font-size: 0.85rem;
  color: #888;
  line-height: 1.3;
}

.timer-info div {
  margin: 0.1rem 0;
  white-space: nowrap;
}

/* 現代化按鈕設計 */
.btn-modern {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 0.75rem 1.25rem;
  font-size: 0.9rem;
  font-weight: 600;
  border: none;
  border-radius: 10px;
  min-width: 120px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  position: relative;
  overflow: hidden;
}

.btn-modern:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.btn-modern:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.btn-icon {
  font-size: 1.1rem;
  transition: transform 0.3s ease;
}

.btn-modern:hover .btn-icon {
  transform: scale(1.1);
}

.btn-text {
  font-size: 1rem;
  letter-spacing: 0.025em;
}

.reset-btn {
  background: linear-gradient(135deg, #4CAF50, #45a049);
  color: white;
}

.reset-btn:hover {
  background: linear-gradient(135deg, #45a049, #3d8b40);
}

.remove-btn {
  background: linear-gradient(135deg, #f44336, #da190b);
  color: white;
}

.share-btn {
  background: linear-gradient(135deg, #42b883, #35495e);
  color: white;
}

.share-btn:hover {
  background: linear-gradient(135deg, #35495e, #42b883);
}

.remove-btn:hover {
  background: linear-gradient(135deg, #da190b, #c11005);
}

/* 響應式設計 */
@media (max-width: 768px) {
  .timer-card {
    flex-direction: column;
    text-align: center;
    gap: 1rem;
    padding: 1rem;
  }

  .timer-left,
  .timer-center,
  .timer-right {
    flex: none;
    width: 100%;
  }

  .boss-info {
    justify-content: center;
    text-align: left;
  }

  .boss-image {
    width: 50px;
    height: 50px;
  }

  .timer-display {
    font-size: 2rem;
  }

  .timer-info div {
    white-space: normal;
  }

  .timer-right {
    flex-direction: row;
    justify-content: center;
    gap: 0.75rem;
  }

  .btn-modern {
    min-width: 100px;
    padding: 0.6rem 1rem;
  }

  .btn-text {
    display: none;
  }

  .btn-icon {
    font-size: 1.2rem;
  }

  .location-item {
    font-size: 0.8rem;
  }
}
</style>

<!-- 主題相關的全域樣式 -->
<style>
/* 深色主題下的計時器卡片樣式 */
body.dark-theme .timer-card .boss-image {
  border-color: #333;
  background-color: #2a2a2a;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

body.dark-theme .timer-card .boss-image:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
}

/* 深色主題下的計時器狀態邊框 */
body.dark-theme .timer-card.timer-waiting {
  border-color: #61dafb;
}

body.dark-theme .timer-card.timer-ready {
  border-color: #4CAF50;
}

body.dark-theme .timer-card.timer-overdue {
  border-color: #ff6b6b;
}

/* 淺色主題樣式 */
body.light-theme .timer-card {
  background-color: #f9f9f9;
  border-color: #ddd;
}

body.light-theme .timer-card.timer-waiting {
  background-color: #f8f9ff;
  border-color: #0066cc;
}

body.light-theme .timer-card.timer-ready {
  background-color: #f0fff4;
  border-color: #28a745;
  box-shadow: 0 0 10px rgba(40, 167, 69, 0.2);
}

body.light-theme .timer-card.timer-overdue {
  background-color: #fff5f5;
  border-color: #dc3545;
  box-shadow: 0 0 10px rgba(220, 53, 69, 0.2);
}

/* 淺色主題狀態徽章 */
body.light-theme .timer-card.timer-waiting .timer-status-badge {
  background-color: rgba(30, 64, 175, 0.15);
  color: #1e40af;
  border: 2px solid #1e40af;
  font-weight: 600;
}

body.light-theme .timer-card.timer-ready .timer-status-badge {
  background-color: rgba(40, 167, 69, 0.15);
  color: #28a745;
  border: 2px solid #28a745;
  font-weight: 600;
}

body.light-theme .timer-card.timer-overdue .timer-status-badge {
  background-color: rgba(220, 53, 69, 0.15);
  color: #dc3545;
  border: 2px solid #dc3545;
  font-weight: 600;
}

/* 淺色主題下的計時器顯示數字 */
body.light-theme .timer-card .timer-display {
    background-color: #eee;
    border-radius: 8px;
}

body.light-theme .timer-card .timer-display.timer-waiting {
  color: #1e40af;
  text-shadow: none;
  font-weight: 700;
}

body.light-theme .timer-card .timer-display.timer-ready {
  color: #28a745;
  text-shadow: none;
}

body.light-theme .timer-card .timer-display.timer-overdue {
  color: #dc3545;
  text-shadow: none;
}

/* 淺色主題下的文字顏色 */
body.light-theme .timer-card .boss-name {
  color: #e65100;
}

body.light-theme .timer-card .channel-name {
  color: #495057;
}

body.light-theme .timer-card .location {
  color: #6c757d;
}

body.light-theme .timer-card .timer-info {
  color: #6c757d;
}

/* 淺色主題下的 BOSS 圖片樣式 */
body.light-theme .timer-card .boss-image {
  border-color: #ddd;
  background-color: #ffffff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

body.light-theme .timer-card .boss-image:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
</style>
