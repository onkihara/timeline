<!-- eslint-disable vue/multi-word-component-names -->
<script setup>

import { ref, reactive, computed, onBeforeMount, onMounted, watch } from 'vue'
import { items } from "../Data.js"
import TimelineItems from "./TimelineItems.vue"

const props = defineProps({
  // props-style
  height: { type: String, default: '100%' },
  width : { type: String, default: '100%' },
  // props-data
  days : { type: Number, default: 5 },
  // Unix-UTC-Timestamp (!)
  startAtDay : { type: Object, default() { return new Date }},
  startAtMonday : { type : Boolean, default: false },
  // day-default
  fullWeek : { type: Boolean, default: false },
  // hour-default
  startAtHour : { type: Number, default: 8 },
  hours : { type: Number, default: 10 },
  // locale options
  locale : { type: String, default: "de-DE" },
  intl : { type: Object, default() { return {  }}},
  // date: short, long or numeric, time: numeric, 2-digit
  dateOptions : { type: Object, default() {return {weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' }}},
  timeOptions : { type: Object, default() { return { hour: '2-digit', minute: '2-digit'}}}
})

// period category
const period = ref('days')
// start date of period
const start = ref(props.startAtDay)
// end date of period
const end = ref(null)


const data = reactive({
  days: props.days,
  hours: props.hours,
  sectors: [],
  items : []
})

const dimensions = reactive({
  timelineWidth : 0,
  sectorWidth : 0,
  firstWidth : 0
})


onBeforeMount(() => {
  data.days = props.fullWeek ? 7 : props.days
  data.hours = props.hours
  data.items = getData()
  calcDays()
})


onMounted(() => {
  new ResizeObserver(calcSectorDimensions)
    .observe(document.querySelector('.v-timeline'))
  calcSectorDimensions()
})


const startDate = computed({
  get() {
    return start.value.toISOString().split('T')[0]
  },
  set(value) {
    start.value = new Date(value)
  }
})


watch(start, (newStart) => {
  start.value = newStart
  calcHeader()
  if (period.value === 'days') {
    calcDays()
  }
  if (period.value === 'hours') {
    calcHours()
  }
})


function calcSectorDimensions() {
  const cont = document.querySelector('.v-timeline-container')
  dimensions.timelineWidth = cont?.clientWidth
  if (dimensions.timelineWidth) {
    dimensions.sectorWidth = Math.floor(dimensions.timelineWidth / (data.sectors.length + 1))
    dimensions.firstWidth = dimensions.timelineWidth - dimensions.sectorWidth * data.sectors.length
  }
}


function hoursOfDay(timedata) {
  start.value = new Date(timedata.date)
  period.value = 'hours'
  calcHours()
}


function daysOfWeek() {
  period.value = 'days'
  calcDays()
}


function setEndOfPeriod() {
  const date = new Date(data.sectors[data.sectors.length-1].date)
  date.setDate(date.getDate()+1)
  date.setSeconds(date.getSeconds()-1)
  end.value = date
}


function next() {
  if (period.value === 'days') {
    start.value.setDate(start.value.getDate() + data.days)
    calcDays()
  }
  if (period.value === 'hours') {
    start.value.setDate(start.value.getDate() + 1)
    calcHours()
  }
  startDate.value = start.value
}


function previous() {
  if (period.value === 'days') {
    start.value.setDate(start.value.getDate() - data.days)
    calcDays()
  }
  if (period.value === 'hours') {
    start.value.setDate(start.value.getDate() - 1)
    calcHours()
  }
  startDate.value = start.value
}

function calcHours() {
  let day = start.value
  let offset = props.startAtHour

  const hours = []
  for (let i = 0; i < data.hours; i++) {
    day.setHours(offset + i)
    hours.push({
      hour : day.getHours(),
      time : day.toLocaleTimeString(props.locale, props.timeOptions),
      isodate : day.toISOString().split('T')[0],
      date : new Date(start.value.setHours(day.getHours()))
    })
  }
  data.sectors = hours
  setEndOfPeriod()
}


function calcDays() {
  // full week?
  let first = start.value
  if (props.startAtMonday) {
    first = getMonday(first)
  }
  // calc days of period
  const days = []
  for (let i = 0; i < data.days; i++) {
    const date = new Date(first)
    date.setDate(date.getDate() + i)
    days.push({
      weekday : date.toLocaleDateString(props.locale, { weekday: props.dateOptions.weekday}),
      day : date.getDate(),
      month : date.toLocaleDateString(props.locale,{ month: props.dateOptions.month }),
      year : date.getFullYear(),
      date : date
    })
  }
  data.sectors = days
  setEndOfPeriod()
}


function calcHeader() {
  calcSectorDimensions()
  // days header
  if (period.value === 'days') {
    // get month of start
    const first = data.sectors.at(0).month
    const firstYear = data.sectors.at(0).year
    // get month of end
    const last = data.sectors.at(-1).month
    const lastYear = data.sectors.at(-1).year
    // compare and combine if not eqal
    if (firstYear == lastYear) {
      return first === last ? first + ' ' + firstYear : first + ' - ' + last + ' ' + firstYear
    } else {
      return first + ' ' + firstYear + ' - ' + last + ' ' + lastYear
    }
  }
  // hours header
  if (period.value === 'hours') {
    return start.value.toLocaleDateString(props.locale, props.dateOptions)
  }

}



function getMonday(d) {
  let day = d.getDay()
  let diff = d.getDate() - day + (day == 0 ? -6 : 1);
  const date = new Date(d.setDate(diff))
  return date;
}

function getData() {
  return items // meanwhile -> todo: fetch json-data
}



</script>




<template>
  <div class="v-timeline"
    :style="{ height: props.height, width: props.width }"
  >

    <div class="v-timeline-month-year-header">

      <a href="" v-on:click.prevent="previous">
        <slot name="previous">Previous</slot>
      </a>
      <div class="v-timeline-month-year">
        {{ calcHeader() }}
        <input
          type="date"
          v-model="startDate"
        >
        <span v-if="period === 'hours'">
          <a href="" v-on:click.prevent="daysOfWeek">
            <slot name="week">Week</slot>
          </a>
        </span>
      </div>
      <a href="" v-on:click.prevent="next">
        <slot name="next">Next</slot>
      </a>
    </div>

    <div
      class="v-timeline-container v-timeline-container-header"
      ref="timeline-container"
    >

      <div
        class="v-timeline-item v-timeline-header"
        :style="{ width : dimensions.firstWidth+'px' }"
      ></div>

      <div class="v-timeline-item v-timeline-header"
        :style="{ width : dimensions.sectorWidth+'px' }"
        :data-date="ts.date"
        :data-sector="i"
        v-for="(ts, i) in data.sectors"
        :key="i"
      >
        <a class="v-timeline-header-content" v-if="period === 'days'" href="" v-on:click.prevent="hoursOfDay(ts)">
          <span class="v-timeline-day">{{ ts.weekday }}<br />{{ ts.day }}</span>
        </a>
        <div class="v-timeline-header-content" v-if="period === 'hours'">
          <span class="v-timeline-hour">{{ ts.time }}</span>
        </div>
      </div>
    </div>

    <div
      class="v-timeline-container v-timeline-container-sector"
      v-for="(item, k) in data.items"
      :key="k"
    >

      <div
        class="v-timeline-item v-timeline-group"
        :style="{ width : dimensions.firstWidth+'px' }"
      >
        {{ item.group }}
      </div>

      <div class="v-timeline-item v-timeline-sector"
      :style="{ width : dimensions.sectorWidth+'px' }"
        :data-date="ts.date"
        :data-sector="i"
        v-for="(ts, i) in data.sectors"
        :key="i"
      >
      &nbsp;
      </div>

      <TimelineItems
        :dates="item.dates"
        :start="start"
        :end="end"
        :period="period"
        :dimensions="dimensions"
        :sectors="data.sectors"
        :hours="{ start : props.startAtHour, number : props.hours }"
      />

    </div>

  </div>
</template>




<style scoped>

  .v-timeline {

    .v-timeline-month-year-header {
      display:flex;
      justify-content: space-between;
      text-align:center;
      padding:10px;
      background-color:beige;
      border-top: 1px solid #FFF;

      .v-timeline-month-year {
        input {
          border:0;
          background:transparent;
          width:20px;
        }
      }
    }

    .v-timeline-container {

      &.v-timeline-container-header {
        height:70px;
      }

      .v-timeline-item {
        display: block;
        float:left;
        box-sizing: border-box;

        &:nth-child(odd) {
          background-color: antiquewhite;
        }

        &:nth-child(even) {
          background-color: aliceblue;
        }

      }

      .v-timeline-header {
        height: 100%;
        text-align:center;
        border-bottom:1px solid #FFF;
        border-top: 1px solid #FFF;

        .v-timeline-header-content {
          height:100%;
          width:100%;
          display:flex;
          justify-content: center;
          align-items: center;
        }

      }
    }

    .v-timeline-container-sector {

      position: relative;
      height: 50px;

      .v-timeline-item {
        height: 50px;
        border-bottom: 1px solid #FFF;
      }

      .v-timeline-group {
        display:flex;
        align-items: center;
        padding-left:10px;
      }
    }


  }

</style>
