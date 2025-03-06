!-- eslint-disable vue/multi-word-component-names -->
<script setup>

import { ref, reactive, computed, onBeforeMount, onMounted, watch } from 'vue'



const props = defineProps({
  dates: { type: Array, default: function() { return [] }},
  start : { type: Object, default: function() { return new Date }},
  end : { type: Object, default: function() { return new Date }},
  period : { type: String, default: 'days' },
  dimensions : { type: Object, default: function() { return {}}},
  sectors : { type: Array, default: function() { return [] }},
  hours : { type: Object, default: function() { return { start : 8, number : 8 }}},
  hourOffset : { type: Object, default: function() { return { right : 1, left : 1 }}},
})

onMounted(() => {

})

const dateEntries = computed(() => {
  return props.dates.filter((date) => isInTimespan(date))
    .map((date) => setPosition(date))
})


function setPosition(date) {
  date.position = findSectorPosition(date)
  return date
}

// calculating with millisecondes UTC
function findSectorPosition(date) {
  const pos = {
    left : props.dimensions.firstWidth,
    right : props.dimensions.firstWidth,
    width : 0,
    lopen : true,
    ropen : true
  }
  const start = new Date(date.start).getTime()
  const end = new Date(date.end).getTime()
  for (let i = 0; i < props.sectors.length; i++) {
    const sectorTime = props.sectors[i].date.getTime()
    if (start >= sectorTime + getADiff()) {
      pos.left += props.dimensions.sectorWidth
      pos.lopen = false
    } else if (end >= sectorTime) {
        pos.width += props.dimensions.sectorWidth
    }
  }
  pos.ropen = isEndPosition(end, props.sectors[0].date)
  pos.right = pos.left + pos.width
  if (props.period === 'days') {
    findHourOffset(pos, date)
  }
  if (props.period === 'hours') {
    findMinuteOffset(pos, date)
  }
  return pos
}


function findHourOffset(pos, date) {
  let startH = new Date(date.start).getHours()
  let leftDiff = 0
  let hourEnd = props.hours.start + props.hours.number
  // start only if not out of range
  if ( ! pos.lopen) {
    if (new Date(date.start).getMinutes() > 30) {
      startH += 1
    }
    if (startH < props.hours.start) {
      startH = props.hours.start
    } else if (startH > hourEnd) {
      startH = hourEnd
    }
    // offset left
    if (startH == hourEnd) {
      startH -= props.hourOffset.left
    }
    leftDiff = Math.floor(props.dimensions.sectorWidth * (startH - props.hours.start) / props.hours.number)
  }
  pos.left += leftDiff
  // end only if not out of rangerops.hours.number
  let endH = new Date(date.end).getHours()
  let rightDiff = 0
  if ( ! pos.ropen) {
    if (new Date(date.end).getMinutes() > 30) {
      endH += 1
    }
    if (endH < props.hours.start) {
      endH = props.hours.start
    } else if (endH > hourEnd) {
      endH = hourEnd
    }
    // offset right
    if (endH == props.hours.start) {
      endH += props.hourOffset.right
    }
    rightDiff = props.dimensions.sectorWidth - Math.floor(props.dimensions.sectorWidth * (endH - props.hours.start) / props.hours.number)
  }
  pos.right -= rightDiff
  pos.width = pos.right - pos.left
}


function findMinuteOffset(pos, date) {
  let startM = new Date(date.start).getMinutes()
  let leftDiff = 0
  // start only if not out of range
  if ( ! pos.lopen) {
    leftDiff = Math.floor(props.dimensions.sectorWidth * startM / 60)
  }
  pos.left += leftDiff
  if (pos.left >= props.dimensions.timelineWidth) {
    pos.left = props.dimensions.timelineWidth - 3
  }
  // end only if not out of range
  let endM = new Date(date.end).getMinutes()
  let rightDiff = 0
  if ( ! pos.ropen && endM > 0) {
    rightDiff = props.dimensions.sectorWidth - Math.floor(props.dimensions.sectorWidth * endM / 60)
  }
  pos.right -= rightDiff
  pos.width = pos.right - pos.left
}



function isInTimespan(date) {
  const start = new Date(date.start)
  const end = new Date(date.end)
  let pstart, pend
  // week-days
  if (props.period === 'days') {
    pstart = props.start
    pend = props.end
  }
  // day-hours
  if (props.period === 'hours') {
    pstart = new Date(props.start.setHours(0))
    pend = new Date(props.start.setHours(0) + (24 * 60 * 60 * 1000) - 1)
  }
  return (start >= pstart && start <= pend)
    || (end > pstart && end <= pend)
    || (start < pstart && end > pend)
}


function getADiff() {
  if (props.period === 'days') {
    return 24*60*60*1000
  }
  if (props.period === 'hours') {
    return 60*60*1000
  }
  return 0
}


function isEndPosition(end, dayDate) {
  if (props.period === 'days' && end <= props.end.getTime() + getADiff()) {
    return false
  }
  if (props.period === 'hours' && end <= dayDate.setHours(props.hours.start + props.hours.number) ) {
    return false
  }
  return true
}


</script>


<template>

    <div
      :id="'v-timeline-date-'+date.id"
      class="v-timeline-date"
      :class="{
        lopen : date.position.lopen,
        ropen : date.position.ropen
      }"
      :style="{
        left : date.position.left + 'px',
        width : date.position.width + 'px'}"
      v-for="date in dateEntries"
      :key="date.id"
    >
      <div class="v-timeline-entry">
        {{ date.start }} - {{ date.end }}
      </div>
    </div>

</template>


<style scoped>

.v-timeline-date {
  position: absolute;
  top:15%;
  height:70%;

  .v-timeline-entry {
    box-sizing: border-box;
    display:flex;
    align-items:center;
    background-color: rgba(252, 119, 119, 0.5);
    height: 100%;
  }

  &.lopen .v-timeline-entry {
    border-left: 4px dotted red;
  }

  &.ropen .v-timeline-entry {
    border-right: 4px dotted red;
  }

}

</style>
