<!-- eslint-disable vue/multi-word-component-names -->
<script setup>

import { ref, reactive, computed, onMounted, watch } from 'vue'

const props = defineProps({

  // props-style
  height: { type: String, default: '100%' },
  width : { type: String, default: '100%' },

  // props-data
  zoom : { type: Number, default: 10 },
  // Unix-UTC-Timestamp (!)
  start : { type: Object, default: Date.now() },

  // props-default
  minZoom : { type: Number, default: 1 },
  maxZoom : { type: Number, default: 28 },
  locale : { type: String, default: "de-DE" },
  fullWeek : { type: Boolean, default: true },

  // short, long or numeric
  dayFormat : { type: String, default: 'long' },
  monthFormat : { type: String, default: 'long' },

})

const period = ref('days')

const data = reactive({

  zoom: props.zoom,
  start: props.start,


})


onMounted(() => {
  //console.log(props.start, data.start.toISOString())
})


const sectors = computed(() => {
  if (period.value === 'hours') {
    return Math.floor(data.zoom*2)
  }

  // days-Sectors
  return calcDays()
})


function zoomHandler(event) {
  if(event.deltaY < 0) {
    if (data.zoom == props.minZoom) {
      return
    }
    data.zoom -= 1
  } else {
    if (data.zoom == props.maxZoom) {
      return
    }
    data.zoom += 1
  }
  if (data.zoom <= 4) {
    period.value = 'hours'
  } else {
    period.value = 'days'
  }
}


function calcDays() {
  // week starts on Monday?
  let start;
  if (props.fullWeek) {
    start = getMonday(data.start)
  } else {
    start = data.start
  }
  // calc days of period
  const days = []
  for (let i = 0; i < data.zoom/2; i++) {
    const date = new Date(start)
    date.setDate(date.getDate() + i)
    days.push({
      weekday : date.toLocaleDateString(props.locale, { weekday: props.dayFormat}),
      day : date.getDate(),
      month : date.toLocaleDateString(props.locale,{ month: props.monthFormat }),
      year : date.getFullYear(),
    })
  }
  return days
}


function calcMonth() {
  // get month of start
  const first = sectors.value.at(0).month
  const firstYear = sectors.value.at(0).year
  // get month of end
  const last = sectors.value.at(-1).month
  const lastYear = sectors.value.at(-1).year
  // compare and combine if not eqal
  if (firstYear == lastYear) {
    return first === last ? first + ' ' + firstYear : first + ' - ' + last + ' ' + firstYear
  } else {
    return first + ' ' + firstYear + ' - ' + last + ' ' + lastYear
  }
}


function getMonday(d) {
  d = new Date(d)
  var day = d.getDay(),
    diff = d.getDate() - day + (day == 0 ? -6 : 1);
  return new Date(d.setDate(diff));
}



</script>




<template>
  <div class="v-timeline"
    :style="{ height: props.height, width: props.width }"
    v-on:wheel="zoomHandler">

    <div
      class="v-timeline-month-year-header">

      <div class="v-timeline-month-year">
        {{ calcMonth() }}
        <input type="date" v-value="data.start.toISOString().split('T')[0]" v-on:input="data.start = new Date($event.target.value)">
      </div>
    </div>

    <div
      class="v-timeline-container v-timeline-container-header">

      <div class="v-timeline-item v-timeline-header"
        v-for="(ts, i) in sectors"
        :key="i">
        <span class="v-timeline-weekday">{{ ts.weekday }}</span>
        <span class="v-timeline-day">{{ ts.day }}</span>
      </div>
    </div>

    <div
      class="v-timeline-container v-timeline-container-sector">

      <div class="v-timeline-item v-timeline-sector"
        v-for="(ts, i) in sectors"
        :key="i">
      </div>
    </div>

  </div>
</template>




<style scoped>

  .v-timeline {
    display: flex;
    flex-direction: column;

    .v-timeline-month-year-header {
      text-align:center;
      padding:10px;
      background-color:beige;
      border: 1px solid black;
      border-bottom: none;

      .v-timeline-month-year {
        input {
          border:0;
          background:transparent;
          width:20px;
        }
      }
    }

    .v-timeline-container {
      display:flex;
      flex-direction: row;

      .v-timeline-item {
        border-left: 1px solid black;
        flex: 1;
        min-width:0;

        &:nth-child(odd) {
          background-color: antiquewhite;
        }

        &:nth-child(even) {
          background-color: aliceblue;
        }

        &:last-child {
          border-right: 1px solid black;
        }
      }

      .v-timeline-header {
        text-align:center;
        padding:10px;
        border-bottom:1px solid black;
        border-top: 1px solid black;

        .v-timeline-weekday, .v-timeline-day {
          display:block;
        }
      }
    }

    .v-timeline-container-sector {
      height: 100vh;
    }
  }

</style>
