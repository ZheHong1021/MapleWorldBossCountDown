<template>
  <div class="add-timer-form">
    <h2>新增 BOSS 倒數計時</h2>
    
    <h3>📋 手動新增 BOSS 計時器</h3>
    
    <div class="form-row">
      <div class="form-group">
        <label for="boss-select">選擇 BOSS：</label>
        <select id="boss-select" v-model="selectedBoss">
          <option value="">請選擇 BOSS</option>
          <option v-for="boss in bosses" :key="boss.id" :value="boss">
            {{ boss.name }} ({{ formatRespawnTime(boss.minRespawnMinutes) }}~{{ formatRespawnTime(boss.maxRespawnMinutes) }}) - {{ formatLocations(boss.location) }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label for="channel-input">頻道名稱：</label>
        <input 
          id="channel-input" 
          type="text" 
          v-model="channelName" 
          placeholder="例如：1168頻道"
        >
      </div>
    </div>

    <div class="form-group" style="display: flex; align-items: center; gap: 0.5rem;">
      <input 
        type="checkbox" 
        id="use-custom-time" 
        v-model="useCustomTime"
        style="margin: 0;"
      >
      <label for="use-custom-time" style="cursor:pointer; margin: 0;">
        自訂開始時間 (不勾選則使用當前時間)
      </label>
    </div>

    <div v-if="useCustomTime === true" :key="useCustomTime" class="form-group custom-time-group">
      <label for="custom-time">開始時間：</label>
      <div class="custom-time-container">
        <div class="datetime-input-wrapper">
          <input 
            id="custom-time" 
            type="datetime-local" 
            v-model="customTime"
            ref="customTimeInput"
            class="custom-time-input"
            placeholder="點擊右側日曆圖標選擇時間"
            title="點擊右側日曆圖標開啟日期時間選擇器"
          >
          <div class="datetime-hint">
            💡 點擊右側的圖標來選擇日期和時間
          </div>
        </div>
        <div class="quick-time-buttons">
          <button type="button" @click="setQuickTime(-1)" class="quick-btn">1分鐘前</button>
          <button type="button" @click="setQuickTime(-3)" class="quick-btn">3分鐘前</button>
          <button type="button" @click="setQuickTime(-10)" class="quick-btn">10分鐘前</button>
          <button type="button" @click="setQuickTime(-30)" class="quick-btn">30分鐘前</button>
          <button type="button" @click="setQuickTime(0)" class="quick-btn">現在</button>
        </div>
      </div>
      <small class="time-hint">💡 使用快速選擇按鈕更方便，或點擊上方時間框右側的日曆圖標精確設定</small>
    </div>

    <div class="button-container">
      <button @click="handleAddTimer" :disabled="!selectedBoss || !channelName" class="start-timer-btn">
        <span class="btn-icon">🚀</span>
        <span class="btn-text">開始倒數計時</span>
      </button>
    </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue'

export default {
  name: 'TimerForm',
  props: {
    bosses: {
      type: Array,
      required: true
    }
  },
  emits: ['add-timer'],
  setup(props, { emit }) {
    const selectedBoss = ref('')
    const channelName = ref('')
    const useCustomTime = ref(false)
    const customTime = ref('')

    // 監聽自訂時間勾選狀態，取消勾選時清空時間值
    watch(useCustomTime, (newValue) => {
      if (!newValue) {
        customTime.value = ''
      }
    })

    // 格式化位置顯示
    const formatLocations = (locations) => {
      if (Array.isArray(locations)) {
        if (locations.length === 1) {
          return locations[0]
        } else if (locations.length > 1) {
          return `${locations[0]} 等${locations.length}個位置`
        }
        return '未知位置'
      }
      return locations || '未知位置'
    }

    // 格式化重生時間範圍 (用於選單顯示)
    const formatRespawnTime = (minutes) => {
      const hours = Math.floor(minutes / 60)
      const remainingMinutes = minutes % 60
      
      if (hours === 0) {
        return `${minutes}分`
      } else if (remainingMinutes === 0) {
        return `${hours}小時`
      } else {
        return `${hours}小時${remainingMinutes}分`
      }
    }

    // 設定快速時間選擇
    const setQuickTime = (minutesOffset) => {
      const now = new Date()
      const targetTime = new Date(now.getTime() + minutesOffset * 60 * 1000)
      
      // 格式化為 datetime-local 輸入框所需的格式
      const year = targetTime.getFullYear()
      const month = String(targetTime.getMonth() + 1).padStart(2, '0')
      const day = String(targetTime.getDate()).padStart(2, '0')
      const hours = String(targetTime.getHours()).padStart(2, '0')
      const minutes = String(targetTime.getMinutes()).padStart(2, '0')
      
      customTime.value = `${year}-${month}-${day}T${hours}:${minutes}`
    }

    // 處理新增計時器
    const handleAddTimer = () => {
      if (!selectedBoss.value || !channelName.value) return

      const timerData = {
        boss: selectedBoss.value,
        channelName: channelName.value,
        useCustomTime: useCustomTime.value,
        customTime: customTime.value
      }

      emit('add-timer', timerData)

      // 清空表單
      selectedBoss.value = ''
      channelName.value = ''
      useCustomTime.value = false
      customTime.value = ''
    }

    return {
      selectedBoss,
      channelName,
      useCustomTime,
      customTime,
      formatLocations,
      formatRespawnTime,
      setQuickTime,
      handleAddTimer
    }
  }
}
</script>

<style scoped>
/* 表單排版樣式 */
.add-timer-form {
  padding: 2rem;
  border-radius: 12px;
  margin: 2rem 0;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
}


.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-group label {
  font-weight: 600;
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.9rem;
  margin-bottom: 0.25rem;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
  font-size: 1rem;
  transition: all 0.3s ease;
  box-sizing: border-box;
}

.form-group input:focus,
.form-group select:focus {
  outline: none;
  border-color: #61dafb;
  box-shadow: 0 0 0 2px rgba(97, 218, 251, 0.2);
  background: rgba(255, 255, 255, 0.15);
}

/* 自訂時間選擇器樣式 */
.custom-time-group {
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 1rem;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.05);
}

.custom-time-container {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.datetime-input-wrapper {
  position: relative;
}

.custom-time-input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
  font-size: 1rem;
}

.datetime-hint {
  font-size: 0.75rem;
  margin-top: 0.25rem;
  font-style: italic;
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.quick-time-buttons {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.quick-btn {
  padding: 0.4rem 0.8rem;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

/* 深色主題樣式 */
body.dark-theme .quick-btn {
  border: 1px solid rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.1);
}

body.dark-theme .quick-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.5);
}

/* 淺色主題樣式 */
body.light-theme .quick-btn {
  border: 2px solid #bbb;
  background: white;
  color: #2d3748;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

body.light-theme .quick-btn:hover {
  background: #f8f9fa;
  border-color: #646cff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

.time-hint {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.8rem;
  font-style: italic;
  transition: color 0.3s ease;
}

/* 按鈕容器樣式 */
.button-container {
  display: flex;
  justify-content: center;
  width: 100%;
  margin-top: 2rem;
}

/* 開始計時按鈕樣式 */
.start-timer-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 1rem 2rem;
  margin: 0;
  font-size: 1.1rem;
  font-weight: 700;
  border: none;
  border-radius: 50px;
  min-width: 200px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  
  /* 可用狀態 - 醒目的漸層背景 */
  background: linear-gradient(135deg, #ff6b6b, #ffa500, #ff6b6b);
  background-size: 200% 200%;
  color: white;
  box-shadow: 0 8px 25px rgba(255, 107, 107, 0.4);
  animation: gradient-shift 3s ease infinite;
}

.start-timer-btn:not(:disabled):hover {
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 12px 35px rgba(255, 107, 107, 0.6);
  background: linear-gradient(135deg, #ff5252, #ff9800, #ff5252);
}

.start-timer-btn:not(:disabled):active {
  transform: translateY(-1px) scale(1.01);
  box-shadow: 0 6px 20px rgba(255, 107, 107, 0.5);
}

/* 禁用狀態 - 明顯的灰色樣式 */
.start-timer-btn:disabled {
  background: linear-gradient(135deg, #424242, #616161);
  color: #9e9e9e;
  cursor: not-allowed;
  transform: none;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  animation: none;
  opacity: 0.6;
}

.start-timer-btn:disabled:hover {
  transform: none;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.start-timer-btn:disabled .btn-icon {
  opacity: 0.5;
  transform: none;
}

.start-timer-btn:disabled .btn-text {
  color: #9e9e9e;
}

/* 漸層動畫 */
@keyframes gradient-shift {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

/* 按鈕圖標動畫 */
.start-timer-btn:not(:disabled) .btn-icon {
  animation: rocket-bounce 2s ease-in-out infinite;
}

@keyframes rocket-bounce {
  0%, 100% {
    transform: translateY(0) rotate(0deg);
  }
  50% {
    transform: translateY(-2px) rotate(5deg);
  }
}

/* 淺色主題樣式 */
body.light-theme .add-timer-form {
  background: #f8f9fa;
  border-color: #e9ecef;
}


body.light-theme .form-group label {
  color: #495057;
  font-weight: 600;
}

body.light-theme .form-group input,
body.light-theme .form-group select {
  background: white;
  border-color: #ced4da;
  color: #495057;
}

body.light-theme .form-group input:focus,
body.light-theme .form-group select:focus {
  border-color: #646cff;
  box-shadow: 0 0 0 2px rgba(100, 108, 255, 0.2);
  background: white;
}

body.light-theme .custom-time-group {
  border-color: #ddd;
  background: #f9f9f9;
}

body.light-theme .custom-time-input {
  border: 2px solid #ddd;
  background: white;
  color: #213547;
}

body.light-theme .custom-time-input:focus {
  border-color: #646cff;
  box-shadow: 0 0 0 3px rgba(100, 108, 255, 0.1);
}

/* 淺色主題下的日期時間選擇器樣式 */
body.light-theme input[type="datetime-local"]::-webkit-calendar-picker-indicator {
  filter: invert(0);
  opacity: 0.8;
  cursor: pointer;
}

body.light-theme input[type="datetime-local"]::-webkit-calendar-picker-indicator:hover {
  opacity: 1;
  background-color: rgba(100, 108, 255, 0.1);
  border-radius: 4px;
}

body.light-theme .quick-time-buttons .quick-btn {
  border: 2px solid #ccc;
  background: white;
  color: #213547;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

body.light-theme .quick-time-buttons .quick-btn:hover {
  background: #f0f0f0;
  border-color: #646cff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

/* 額外的樣式確保快速按鈕在淺色主題下可見 */
body.light-theme button.quick-btn {
  border: 2px solid #bbb;
  background-color: #ffffff;
  color: #2d3748;
}

body.light-theme button.quick-btn:hover {
  border-color: #646cff;
  background-color: #f8f9fa;
}

body.light-theme .datetime-hint {
  font-weight: 500;
}

body.light-theme .time-hint {
  font-weight: 500;
}

body.light-theme .start-timer-btn:not(:disabled) {
  background: linear-gradient(135deg, #007bff, #0056b3, #007bff);
  box-shadow: 0 8px 25px rgba(0, 123, 255, 0.4);
  color: white;
}

body.light-theme .start-timer-btn:not(:disabled):hover {
  background: linear-gradient(135deg, #0056b3, #003d82, #0056b3);
  box-shadow: 0 12px 35px rgba(0, 123, 255, 0.6);
}

body.light-theme .start-timer-btn:disabled {
  background: linear-gradient(135deg, #e9ecef, #dee2e6);
  color: #6c757d;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 響應式設計 */
@media (max-width: 768px) {
  .form-row {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  
  .start-timer-btn {
    min-width: 180px;
    padding: 0.9rem 1.5rem;
    font-size: 1rem;
  }
  
  .start-timer-btn .btn-text {
    font-size: 0.95rem;
  }
}
</style>
