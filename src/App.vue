<script setup>
import Timeline from './components/Timeline.vue'
</script>



<template>
  <div class="timeline-container">

    <Timeline
      height="50vh"
      :start-at-day="new Date('2022-12-28T00:00:00+0100')"
      :start-at-monday="true"
      :full-week="true"
    >
      <template #week>
        <span>Woche</span>
      </template>

    </Timeline>

  </div>
</template>



<style scoped>
  .timeline-container {
    background-color: #f0f0f0;
  }
</style>
