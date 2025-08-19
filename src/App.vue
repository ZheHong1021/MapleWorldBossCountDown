<template>
  <div class="app" :class="{ 'dark-theme': isDarkTheme, 'light-theme': !isDarkTheme }">
    <TopControls 
      :current-time="currentTime"
      :is-dark-theme="isDarkTheme"
      @toggle-theme="toggleTheme"
      @open-import-dialog="showImportDialog = true"
    />

    <h1>🍁 楓之谷世界王倒數計時器</h1>

    <TimerForm 
      :bosses="bosses"
      @add-timer="addTimer"
    />

    <TimerList 
      :timers="activeTimers"
      @reset-timer="resetTimer"
      @remove-timer="removeTimer"
      @share-timer="openShareDialog"
    />

    <!-- 分享對話框 -->
    <ShareDialog
      :is-visible="showShareDialog"
      :timer-data="selectedTimerForShare"
      @close="closeShareDialog"
    />

    <!-- 匯入對話框 -->
    <ImportDialog
      :is-visible="showImportDialog"
      @close="showImportDialog = false"
      @import="importTimers"
    />

    <AppFooter />
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'
import TopControls from './components/TopControls.vue'
import TimerForm from './components/TimerForm.vue'
import TimerList from './components/TimerList.vue'
import AppFooter from './components/AppFooter.vue'
import ShareDialog from './components/ShareDialog.vue'
import ImportDialog from './components/ImportDialog.vue'

export default {
  name: 'App',
  components: {
    TopControls,
    TimerForm,
    TimerList,
    AppFooter,
    ShareDialog,
    ImportDialog
  },
  setup() {
    const bosses = ref([])
    const activeTimers = ref([])
    const currentTime = ref('')
    const isDarkTheme = ref(true) // 預設為深色主題
    const showShareDialog = ref(false)
    const showImportDialog = ref(false)
    const selectedTimerForShare = ref(null)
    
    let updateInterval = null

    // 載入 BOSS 資料
    const loadBossData = async () => {
      try {
        const basePath = import.meta.env.BASE_URL
        const dataUrl = `${basePath}data.json`
        const response = await fetch(dataUrl)
        
        const data = await response.json()
        bosses.value = data.bosses
      } catch (error) {
        console.error('載入 BOSS 資料失敗:', error)
      }
    }

    // 主題相關函數
    const initTheme = () => {
      // 從 localStorage 讀取使用者偏好
      const savedTheme = localStorage.getItem('maple-timer-theme')
      if (savedTheme) {
        isDarkTheme.value = savedTheme === 'dark'
      } else {
        // 如果沒有保存的偏好，使用系統偏好
        isDarkTheme.value = !window.matchMedia('(prefers-color-scheme: light)').matches
      }
      updateThemeClass()
    }

    const toggleTheme = () => {
      isDarkTheme.value = !isDarkTheme.value
      localStorage.setItem('maple-timer-theme', isDarkTheme.value ? 'dark' : 'light')
      updateThemeClass()
    }

    const updateThemeClass = () => {
      // 更新 body 的 class 以確保全域主題一致
      document.body.className = isDarkTheme.value ? 'dark-theme' : 'light-theme'
    }

    // 保存計時器到 localStorage
    const saveTimersToStorage = () => {
      try {
        const timersData = activeTimers.value.map(timer => ({
          id: timer.id,
          boss: timer.boss,
          channelName: timer.channelName,
          startTime: timer.startTime.toISOString(),
          minEndTime: timer.minEndTime.toISOString(),
          maxEndTime: timer.maxEndTime.toISOString()
        }))
        localStorage.setItem('maple-timer-data', JSON.stringify(timersData))
      } catch (error) {
        console.error('保存計時器數據失敗:', error)
      }
    }

    // 從 localStorage 載入計時器
    const loadTimersFromStorage = () => {
      try {
        const savedData = localStorage.getItem('maple-timer-data')
        if (savedData) {
          const timersData = JSON.parse(savedData)
          const now = new Date()
          
          // 過濾掉超過24小時的舊計時器
          const validTimers = timersData.filter(timerData => {
            const startTime = new Date(timerData.startTime)
            const hoursSinceStart = (now - startTime) / (1000 * 60 * 60)
            return hoursSinceStart < 24 // 只保留24小時內的計時器
          })

          // 重建計時器對象
          activeTimers.value = validTimers.map(timerData => {
            const startTime = new Date(timerData.startTime)
            const elapsedSeconds = Math.floor((now - startTime) / 1000)
            const totalSeconds = timerData.boss.minRespawnMinutes * 60
            const timeRemaining = Math.max(0, totalSeconds - elapsedSeconds)

            return {
              id: timerData.id,
              boss: timerData.boss,
              channelName: timerData.channelName,
              startTime: startTime,
              minEndTime: new Date(timerData.minEndTime),
              maxEndTime: new Date(timerData.maxEndTime),
              timeRemaining: timeRemaining
            }
          })
        }
      } catch (error) {
        console.error('載入計時器數據失敗:', error)
        // 如果載入失敗，清空無效數據
        localStorage.removeItem('maple-timer-data')
      }
    }

    // 清理過期的計時器（自動執行）
    const cleanupExpiredTimers = () => {
      const now = new Date()
      const beforeCount = activeTimers.value.length
      
      activeTimers.value = activeTimers.value.filter(timer => {
        // 移除超過最大重生時間後1小時的計時器
        const maxEndTimePlus1Hour = new Date(timer.maxEndTime.getTime() + 60 * 60 * 1000)
        return now < maxEndTimePlus1Hour
      })

      // 如果有計時器被清理，保存更新
      if (activeTimers.value.length !== beforeCount) {
        saveTimersToStorage()
      }
    }

    // 更新當前時間
    const updateCurrentTime = () => {
      const now = new Date()
      currentTime.value = now.toLocaleString('zh-TW', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      })
    }

    // 新增計時器
    const addTimer = (timerData) => {
      // 決定使用的開始時間
      let startTime
      if (timerData.useCustomTime && timerData.customTime) {
        startTime = new Date(timerData.customTime)
        // 檢查自訂時間是否有效
        if (isNaN(startTime.getTime())) {
          alert('請輸入有效的日期時間')
          return
        }
      } else {
        startTime = new Date()
      }

      const minEndTime = new Date(startTime.getTime() + timerData.boss.minRespawnMinutes * 60 * 1000)
      const maxEndTime = new Date(startTime.getTime() + timerData.boss.maxRespawnMinutes * 60 * 1000)

      // 計算剩餘時間
      const now = new Date()
      const elapsedSeconds = Math.floor((now - startTime) / 1000)
      const totalSeconds = timerData.boss.minRespawnMinutes * 60
      const timeRemaining = Math.max(0, totalSeconds - elapsedSeconds)

      const timer = {
        id: Date.now() + Math.random(),
        boss: { ...timerData.boss },
        channelName: timerData.channelName,
        startTime: startTime,
        minEndTime: minEndTime,
        maxEndTime: maxEndTime,
        timeRemaining: timeRemaining
      }

      activeTimers.value.push(timer)
      saveTimersToStorage() // 保存到 localStorage
    }

    // 移除計時器
    const removeTimer = (timerId) => {
      const index = activeTimers.value.findIndex(timer => timer.id === timerId)
      if (index !== -1) {
        activeTimers.value.splice(index, 1)
        saveTimersToStorage() // 保存到 localStorage
      }
    }

    // 重新計時
    const resetTimer = (timerId) => {
      const timer = activeTimers.value.find(t => t.id === timerId)
      if (timer) {
        const now = new Date()
        timer.startTime = now
        timer.minEndTime = new Date(now.getTime() + timer.boss.minRespawnMinutes * 60 * 1000)
        timer.maxEndTime = new Date(now.getTime() + timer.boss.maxRespawnMinutes * 60 * 1000)
        timer.timeRemaining = timer.boss.minRespawnMinutes * 60
        saveTimersToStorage() // 保存到 localStorage
      }
    }

    // 更新所有計時器
    const updateTimers = () => {
      const now = new Date()
      let hasExpiredTimers = false
      
      activeTimers.value.forEach(timer => {
        const elapsedSeconds = Math.floor((now - timer.startTime) / 1000)
        const totalSeconds = timer.boss.minRespawnMinutes * 60
        timer.timeRemaining = Math.max(0, totalSeconds - elapsedSeconds)
        
        // 檢查是否過期 (超過最大重生時間 24 小時)
        if (elapsedSeconds > timer.boss.maxRespawnMinutes * 60) {
          hasExpiredTimers = true
        }
      })
      
      // 如果有過期計時器，執行清理
      if (hasExpiredTimers) {
        cleanupExpiredTimers()
      }
    }

    // 啟動更新循環
    const startUpdateLoop = () => {
      updateCurrentTime()
      updateTimers()
      updateInterval = setInterval(() => {
        updateCurrentTime()
        updateTimers()
      }, 1000)
    }

    // 組件掛載時
    onMounted(() => {
      initTheme()
      loadBossData()
      loadTimersFromStorage() // 載入保存的計時器
      startUpdateLoop()
    })

    // 組件卸載時
    onUnmounted(() => {
      if (updateInterval) {
        clearInterval(updateInterval)
      }
    })

    // 打開分享對話框
    const openShareDialog = (timerId) => {
      const timer = activeTimers.value.find(t => t.id === timerId)
      if (timer) {
        selectedTimerForShare.value = timer
        showShareDialog.value = true
      }
    }

    // 關閉分享對話框
    const closeShareDialog = () => {
      showShareDialog.value = false
      selectedTimerForShare.value = null
    }

    // 匯入計時器從 JSON
    const importTimers = (importData) => {
      try {
        let successCount = 0
        let errorCount = 0
        const errors = []

        for (const timerData of importData.timers) {
          try {
            // 尋找對應的 BOSS
            const boss = bosses.value.find(b => 
              b.id === timerData.bossId || 
              b.name === timerData.bossName
            )

            if (!boss) {
              // 如果找不到對應的 BOSS，創建一個基於匯入資料的 BOSS 物件
              const importedBoss = {
                id: timerData.bossId,
                name: timerData.bossName,
                minRespawnMinutes: timerData.minRespawnMinutes,
                maxRespawnMinutes: timerData.maxRespawnMinutes,
                location: timerData.location
              }
              
              // 使用匯入的 BOSS 資料創建計時器
              const startTime = new Date(timerData.startTime)
              const minEndTime = new Date(startTime.getTime() + importedBoss.minRespawnMinutes * 60 * 1000)
              const maxEndTime = new Date(startTime.getTime() + importedBoss.maxRespawnMinutes * 60 * 1000)

              // 計算剩餘時間
              const now = new Date()
              const elapsedSeconds = Math.floor((now - startTime) / 1000)
              const totalSeconds = importedBoss.minRespawnMinutes * 60
              const timeRemaining = Math.max(0, totalSeconds - elapsedSeconds)

              const timer = {
                id: Date.now() + Math.random(),
                boss: importedBoss,
                channelName: timerData.channelName,
                startTime: startTime,
                minEndTime: minEndTime,
                maxEndTime: maxEndTime,
                timeRemaining: timeRemaining
              }

              activeTimers.value.push(timer)
              successCount++
            } else {
              // 找到對應的 BOSS，使用現有資料創建計時器
              const startTime = new Date(timerData.startTime)
              const minEndTime = new Date(startTime.getTime() + boss.minRespawnMinutes * 60 * 1000)
              const maxEndTime = new Date(startTime.getTime() + boss.maxRespawnMinutes * 60 * 1000)

              // 計算剩餘時間
              const now = new Date()
              const elapsedSeconds = Math.floor((now - startTime) / 1000)
              const totalSeconds = boss.minRespawnMinutes * 60
              const timeRemaining = Math.max(0, totalSeconds - elapsedSeconds)

              const timer = {
                id: Date.now() + Math.random(),
                boss: { ...boss },
                channelName: timerData.channelName,
                startTime: startTime,
                minEndTime: minEndTime,
                maxEndTime: maxEndTime,
                timeRemaining: timeRemaining
              }

              activeTimers.value.push(timer)
              successCount++
            }
          } catch (error) {
            console.error('匯入單個計時器失敗:', error)
            errors.push(`${timerData.bossName} (${timerData.channelName}): ${error.message}`)
            errorCount++
          }
        }

        // 保存到 localStorage
        if (successCount > 0) {
          saveTimersToStorage()
        }

        // 顯示結果訊息
        let message = `匯入完成！\n\n✅ 成功：${successCount} 個計時器`
        if (errorCount > 0) {
          message += `\n❌ 失敗：${errorCount} 個計時器`
          if (errors.length > 0) {
            message += `\n\n失敗詳情：\n${errors.join('\n')}`
          }
        }
        
        alert(message)

      } catch (error) {
        console.error('匯入計時器失敗:', error)
        alert(`匯入失敗：${error.message}`)
      }
    }

    return {
      bosses,
      activeTimers,
      currentTime,
      isDarkTheme,
      showShareDialog,
      showImportDialog,
      selectedTimerForShare,
      addTimer,
      removeTimer,
      resetTimer,
      toggleTheme,
      openShareDialog,
      closeShareDialog,
      importTimers
    }
  }
}
</script>

<style scoped>
.app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
  transition: all 0.3s ease;
  width: 100%;
  box-sizing: border-box;
}
</style>
