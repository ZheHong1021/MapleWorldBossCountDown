<template>
  <div v-if="isVisible" class="dialog-overlay" @click.self="closeDialog">
    <div class="dialog-container">
      <div class="dialog-header">
        <h3>📤 分享計時器</h3>
        <button @click="closeDialog" class="close-btn">✕</button>
      </div>

      <div class="dialog-content">
        <div class="timer-info">
          <div class="boss-info">
            <strong>{{ timerData?.boss?.name || '未知BOSS' }}</strong>
            <span class="channel">{{ timerData?.channelName || '未知頻道' }}</span>
          </div>
          <div class="timer-details">
            <div>開始時間：{{ formatDateTime(timerData?.startTime) }}</div>
            <div>重生時間：{{ timerData?.boss?.minRespawnMinutes || 0 }}-{{ timerData?.boss?.maxRespawnMinutes || 0 }} 分鐘</div>
          </div>
        </div>

        <div class="json-section">
          <label for="share-json">分享 JSON 資料：</label>
          <textarea
            id="share-json"
            v-model="shareJson"
            readonly
            rows="8"
            class="json-textarea"
            @click="selectAllText"
          ></textarea>
        </div>

        <div class="dialog-actions">
          <button @click="copyToClipboard" class="copy-btn" :disabled="!shareJson">
            <span class="btn-icon">📋</span>
            <span class="btn-text">複製到剪貼簿</span>
          </button>
          <button @click="closeDialog" class="cancel-btn">
            <span class="btn-text">關閉</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'

export default {
  name: 'ShareDialog',
  props: {
    isVisible: {
      type: Boolean,
      default: false
    },
    timerData: {
      type: Object,
      default: null
    }
  },
  emits: ['close'],
  setup(props, { emit }) {
    const shareJson = ref('')

    // 當 timerData 改變時生成 JSON
    watch(() => props.timerData, (newTimerData) => {
      if (newTimerData) {
        generateShareJson(newTimerData)
      }
    }, { immediate: true })

    // 生成分享 JSON
    const generateShareJson = (timer) => {
      const shareData = {
        version: "1.0",
        shareTime: new Date().toISOString(),
        timers: [{
          bossId: timer.boss.id,
          bossName: timer.boss.name,
          channelName: timer.channelName,
          startTime: timer.startTime.toISOString(),
          minRespawnMinutes: timer.boss.minRespawnMinutes,
          maxRespawnMinutes: timer.boss.maxRespawnMinutes,
          location: timer.boss.location
        }]
      }
      
      shareJson.value = JSON.stringify(shareData, null, 2)
    }

    // 格式化日期時間
    const formatDateTime = (date) => {
      if (!date) return '未知時間'
      return new Date(date).toLocaleString('zh-TW', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      })
    }

    // 選中所有文字
    const selectAllText = (event) => {
      event.target.select()
    }

    // 複製到剪貼簿
    const copyToClipboard = async () => {
      try {
        if (navigator.clipboard && navigator.clipboard.writeText) {
          await navigator.clipboard.writeText(shareJson.value)
          alert('✅ 已複製到剪貼簿！\n可以分享給其他玩家匯入使用。')
        } else {
          // Fallback 方法
          const textArea = document.createElement('textarea')
          textArea.value = shareJson.value
          textArea.style.position = 'fixed'
          textArea.style.top = '-1000px'
          textArea.style.left = '-1000px'
          document.body.appendChild(textArea)
          textArea.select()
          textArea.setSelectionRange(0, 99999)
          
          try {
            document.execCommand('copy')
            alert('✅ 已複製到剪貼簿！\n可以分享給其他玩家匯入使用。')
          } catch (err) {
            alert('❌ 複製失敗，請手動選擇文字複製')
          }
          
          document.body.removeChild(textArea)
        }
      } catch (error) {
        console.error('複製失敗:', error)
        alert('❌ 複製失敗，請手動選擇文字複製')
      }
    }

    // 關閉對話框
    const closeDialog = () => {
      emit('close')
    }

    return {
      shareJson,
      formatDateTime,
      selectAllText,
      copyToClipboard,
      closeDialog
    }
  }
}
</script>

<style scoped>
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(3px);
}

.dialog-container {
  background: #1a1a1a;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  min-width: 500px;
  max-width: 600px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  animation: dialog-appear 0.3s ease-out;
}

@keyframes dialog-appear {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(-20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem 2rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.dialog-header h3 {
  margin: 0;
  color: #42b883;
  font-size: 1.2rem;
}

.close-btn {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.6);
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 50%;
  transition: all 0.3s ease;
}

.close-btn:hover {
  color: #ff6b6b;
  background: rgba(255, 107, 107, 0.1);
}

.dialog-content {
  padding: 2rem;
}

.timer-info {
  background: rgba(66, 184, 131, 0.1);
  border: 1px solid rgba(66, 184, 131, 0.3);
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.boss-info {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.boss-info strong {
  color: #ffa500;
  font-size: 1.1rem;
}

.channel {
  background: rgba(255, 255, 255, 0.1);
  padding: 0.25rem 0.75rem;
  border-radius: 15px;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.8);
}

.timer-details {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
  line-height: 1.5;
}

.json-section {
  margin-bottom: 2rem;
}

.json-section label {
  display: block;
  margin-bottom: 0.75rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.9);
}

.json-textarea {
  width: 100%;
  padding: 1rem;
  border: 1px solid rgba(66, 184, 131, 0.3);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.05);
  color: #61dafb;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 0.9rem;
  resize: vertical;
  box-sizing: border-box;
  transition: all 0.3s ease;
}

.json-textarea:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 2px rgba(66, 184, 131, 0.2);
  background: rgba(255, 255, 255, 0.1);
}

.dialog-actions {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
}

.copy-btn, .cancel-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.copy-btn {
  background: linear-gradient(135deg, #42b883, #35495e);
  color: white;
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.3);
}

.copy-btn:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(66, 184, 131, 0.4);
  background: linear-gradient(135deg, #35495e, #42b883);
}

.copy-btn:disabled {
  background: linear-gradient(135deg, #6c757d, #495057);
  cursor: not-allowed;
  opacity: 0.6;
}

.cancel-btn {
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.cancel-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  color: white;
}

/* 淺色主題樣式 */
body.light-theme .dialog-container {
  background: #ffffff;
  border-color: #e9ecef;
  color: #495057;
}

body.light-theme .dialog-header {
  border-bottom-color: #e9ecef;
}

body.light-theme .dialog-header h3 {
  color: #28a745;
}

body.light-theme .close-btn {
  color: #6c757d;
}

body.light-theme .close-btn:hover {
  color: #dc3545;
  background: rgba(220, 53, 69, 0.1);
}

body.light-theme .timer-info {
  background: rgba(40, 167, 69, 0.1);
  border-color: #28a745;
}

body.light-theme .boss-info strong {
  color: #e65100;
}

body.light-theme .channel {
  background: #e9ecef;
  color: #495057;
}

body.light-theme .timer-details {
  color: #6c757d;
}

body.light-theme .json-section label {
  color: #495057;
}

body.light-theme .json-textarea {
  background: #f8f9fa;
  border-color: #28a745;
  color: #495057;
}

body.light-theme .json-textarea:focus {
  border-color: #20c997;
  box-shadow: 0 0 0 2px rgba(32, 201, 151, 0.2);
  background: white;
}

body.light-theme .copy-btn {
  background: linear-gradient(135deg, #28a745, #20c997);
  box-shadow: 0 4px 12px rgba(40, 167, 69, 0.3);
}

body.light-theme .copy-btn:not(:disabled):hover {
  background: linear-gradient(135deg, #20c997, #28a745);
  box-shadow: 0 6px 16px rgba(40, 167, 69, 0.4);
}

body.light-theme .cancel-btn {
  background: #e9ecef;
  color: #495057;
  border-color: #dee2e6;
}

body.light-theme .cancel-btn:hover {
  background: #dee2e6;
  color: #495057;
}

/* 響應式設計 */
@media (max-width: 768px) {
  .dialog-container {
    min-width: 300px;
    max-width: 90vw;
    margin: 1rem;
  }
  
  .dialog-content {
    padding: 1.5rem;
  }
  
  .dialog-actions {
    flex-direction: column;
  }
  
  .copy-btn, .cancel-btn {
    width: 100%;
    justify-content: center;
  }
}
</style>