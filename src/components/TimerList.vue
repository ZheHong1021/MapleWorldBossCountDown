<template>
  <div class="timers-container">
    <h2>活躍計時器 ({{ timers.length }})</h2>
    
    <div v-if="timers.length === 0" class="no-timers">
      目前沒有活躍的倒數計時器
    </div>

    <div class="timers-grid">
      <TimerCard 
        v-for="timer in timers" 
        :key="timer.id" 
        :timer="timer"
        @reset-timer="$emit('reset-timer', $event)"
        @remove-timer="$emit('remove-timer', $event)"
        @share-timer="$emit('share-timer', $event)"
      />
    </div>
  </div>
</template>

<script>
import TimerCard from './TimerCard.vue'

export default {
  name: 'TimerList',
  components: {
    TimerCard
  },
  props: {
    timers: {
      type: Array,
      required: true
    }
  },
  emits: ['reset-timer', 'remove-timer', 'share-timer']
}
</script>

<style scoped>
.timers-container {
  margin-top: 2rem;
}

.no-timers {
  text-align: center;
  color: #888;
  font-style: italic;
  padding: 2rem;
}

.timers-grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
</style>
