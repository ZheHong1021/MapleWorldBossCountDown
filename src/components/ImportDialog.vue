<template>
  <div v-if="isVisible" class="dialog-overlay" @click.self="closeDialog">
    <div class="dialog-container">
      <div class="dialog-header">
        <h3>📥 匯入計時器</h3>
        <button @click="closeDialog" class="close-btn">✕</button>
      </div>

      <div class="dialog-content">
        <div class="import-instructions">
          <h4>📋 使用說明</h4>
          <ol>
            <li>請將其他玩家分享的 JSON 資料貼到下方文字框</li>
            <li>確認資料正確後點擊「匯入」按鈕</li>
            <li>計時器將會添加到您目前的列表中</li>
          </ol>
        </div>

        <div class="json-section">
          <label for="import-json">貼上 JSON 資料：</label>
          <textarea
            id="import-json"
            v-model="importJson"
            placeholder="請貼上其他玩家分享的計時器 JSON 資料..."
            rows="8"
            class="json-textarea"
          ></textarea>
          
          <div v-if="validationError" class="error-message">
            ❌ {{ validationError }}
          </div>
          
          <div v-if="previewData" class="preview-section">
            <h4>📋 預覽資料</h4>
            <div class="preview-info">
              <div><strong>分享時間：</strong>{{ formatDateTime(previewData.shareTime) }}</div>
              <div><strong>計時器數量：</strong>{{ previewData.timers?.length || 0 }} 個</div>
              <div v-if="previewData.timers?.length > 0">
                <strong>包含 BOSS：</strong>
                <span class="boss-list">
                  {{ previewData.timers.map(t => `${t.bossName} (${t.channelName})`).join('、') }}
                </span>
              </div>
            </div>
          </div>
        </div>

        <div class="dialog-actions">
          <button @click="clearInput" v-if="importJson" class="clear-btn">
            <span class="btn-icon">🗑️</span>
            <span class="btn-text">清空</span>
          </button>
          <button @click="importTimers" class="import-btn" :disabled="!isValidJson || !importJson.trim()">
            <span class="btn-icon">📥</span>
            <span class="btn-text">匯入計時器</span>
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
  name: 'ImportDialog',
  props: {
    isVisible: {
      type: Boolean,
      default: false
    }
  },
  emits: ['close', 'import'],
  setup(props, { emit }) {
    const importJson = ref('')
    const validationError = ref('')
    const previewData = ref(null)

    // 監聽輸入變化，進行驗證
    watch(importJson, (newValue) => {
      validateAndPreview(newValue)
    })

    // 驗證並預覽
    const validateAndPreview = (jsonString) => {
      validationError.value = ''
      previewData.value = null

      if (!jsonString.trim()) {
        return
      }

      try {
        const data = JSON.parse(jsonString.trim())
        
        // 驗證基本格式
        if (!data.version || !data.timers || !Array.isArray(data.timers)) {
          validationError.value = 'JSON 格式不正確：缺少必要的欄位 (version, timers)'
          return
        }
        
        if (data.timers.length === 0) {
          validationError.value = '分享的資料中沒有計時器'
          return
        }

        // 驗證每個計時器
        for (let i = 0; i < data.timers.length; i++) {
          const timer = data.timers[i]
          if (!timer.bossId || !timer.bossName || !timer.channelName || !timer.startTime) {
            validationError.value = `第 ${i + 1} 個計時器資料不完整：缺少必要欄位`
            return
          }

          // 驗證時間格式
          if (isNaN(new Date(timer.startTime).getTime())) {
            validationError.value = `第 ${i + 1} 個計時器的時間格式不正確`
            return
          }
        }

        // 驗證通過，顯示預覽
        previewData.value = data
        
      } catch (error) {
        validationError.value = '無效的 JSON 格式，請檢查資料是否正確'
      }
    }

    // 是否為有效 JSON
    const isValidJson = computed(() => {
      return previewData.value !== null && !validationError.value
    })

    // 格式化日期時間
    const formatDateTime = (dateString) => {
      try {
        return new Date(dateString).toLocaleString('zh-TW', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit',
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit'
        })
      } catch {
        return '無效時間'
      }
    }

    // 清空輸入
    const clearInput = () => {
      importJson.value = ''
      validationError.value = ''
      previewData.value = null
    }

    // 匯入計時器
    const importTimers = () => {
      if (!isValidJson.value || !previewData.value) {
        return
      }

      // 確認匯入
      const timerCount = previewData.value.timers.length
      const bossNames = previewData.value.timers.map(t => t.bossName).join('、')
      const shareTime = formatDateTime(previewData.value.shareTime)
      
      const confirmMessage = `確定要匯入以下計時器嗎？\n\n分享時間：${shareTime}\n計時器數量：${timerCount} 個\nBOSS：${bossNames}\n\n匯入後將會添加到您目前的計時器列表中`
      
      if (confirm(confirmMessage)) {
        emit('import', previewData.value)
        clearInput()
        closeDialog()
      }
    }

    // 關閉對話框
    const closeDialog = () => {
      emit('close')
    }

    // 當對話框打開時清空內容
    watch(() => props.isVisible, (newVisible) => {
      if (newVisible) {
        clearInput()
      }
    })

    return {
      importJson,
      validationError,
      previewData,
      isValidJson,
      formatDateTime,
      clearInput,
      importTimers,
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
  max-width: 700px;
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

.import-instructions {
  background: rgba(66, 184, 131, 0.1);
  border: 1px solid rgba(66, 184, 131, 0.3);
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.import-instructions h4 {
  margin: 0 0 1rem 0;
  color: #42b883;
  font-size: 1rem;
}

.import-instructions ol {
  margin: 0;
  padding-left: 1.2rem;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
  line-height: 1.6;
}

.import-instructions li {
  margin-bottom: 0.5rem;
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
  color: inherit;
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

.json-textarea::placeholder {
  color: rgba(255, 255, 255, 0.5);
  font-style: italic;
}

.error-message {
  margin-top: 0.75rem;
  padding: 0.75rem;
  background: rgba(255, 107, 107, 0.1);
  border: 1px solid rgba(255, 107, 107, 0.3);
  border-radius: 6px;
  color: #ff6b6b;
  font-size: 0.9rem;
}

.preview-section {
  margin-top: 1rem;
  background: rgba(97, 218, 251, 0.1);
  border: 1px solid rgba(97, 218, 251, 0.3);
  border-radius: 8px;
  padding: 1rem;
}

.preview-section h4 {
  margin: 0 0 0.75rem 0;
  color: #61dafb;
  font-size: 1rem;
}

.preview-info {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
  line-height: 1.6;
}

.preview-info > div {
  margin-bottom: 0.5rem;
}

.preview-info strong {
  color: rgba(255, 255, 255, 0.9);
}

.boss-list {
  color: #ffa500;
  font-weight: 500;
}

.dialog-actions {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
}

.import-btn, .clear-btn, .cancel-btn {
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

.import-btn {
  background: linear-gradient(135deg, #42b883, #35495e);
  color: white;
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.3);
}

.import-btn:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(66, 184, 131, 0.4);
  background: linear-gradient(135deg, #35495e, #42b883);
}

.import-btn:disabled {
  background: linear-gradient(135deg, #6c757d, #495057);
  cursor: not-allowed;
  opacity: 0.6;
}

.clear-btn {
  background: linear-gradient(135deg, #dc3545, #c82333);
  color: white;
  box-shadow: 0 4px 12px rgba(220, 53, 69, 0.3);
}

.clear-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(220, 53, 69, 0.4);
  background: linear-gradient(135deg, #c82333, #dc3545);
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

body.light-theme .import-instructions {
  background: rgba(40, 167, 69, 0.1);
  border-color: #28a745;
}

body.light-theme .import-instructions h4 {
  color: #28a745;
}

body.light-theme .import-instructions ol {
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

body.light-theme .json-textarea::placeholder {
  color: #6c757d;
}

body.light-theme .error-message {
  background: rgba(220, 53, 69, 0.1);
  border-color: #dc3545;
  color: #dc3545;
}

body.light-theme .preview-section {
  background: rgba(13, 202, 240, 0.1);
  border-color: #0dcaf0;
}

body.light-theme .preview-section h4 {
  color: #0dcaf0;
}

body.light-theme .preview-info {
  color: #6c757d;
}

body.light-theme .preview-info strong {
  color: #495057;
}

body.light-theme .boss-list {
  color: #e65100;
}

body.light-theme .import-btn {
  background: linear-gradient(135deg, #28a745, #20c997);
  box-shadow: 0 4px 12px rgba(40, 167, 69, 0.3);
}

body.light-theme .import-btn:not(:disabled):hover {
  background: linear-gradient(135deg, #20c997, #28a745);
  box-shadow: 0 6px 16px rgba(40, 167, 69, 0.4);
}

body.light-theme .clear-btn {
  background: linear-gradient(135deg, #dc3545, #c82333);
  box-shadow: 0 4px 12px rgba(220, 53, 69, 0.3);
}

body.light-theme .clear-btn:hover {
  background: linear-gradient(135deg, #c82333, #dc3545);
  box-shadow: 0 6px 16px rgba(220, 53, 69, 0.4);
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
  
  .import-btn, .clear-btn, .cancel-btn {
    width: 100%;
    justify-content: center;
  }
}
</style>