<script setup>
import Timeline from './components/Timeline.vue'
</script>



<template>
  <div class="timeline-container">

    <Timeline
      height="50vh"
      :start="new Date('2022-12-28')"
    >

    </Timeline>

  </div>
</template>



<style scoped>
  .timeline-container {
    background-color: #f0f0f0;
  }
</style>
