<template>
  <div style="position: relative;">
    <div class="rh-gantt-top-bar">
      <div class="rh-gantt-top-bar-buttons">
        <div><i class="mdi mdi-magnify-minus"></i></div>
        <div><i class="mdi mdi-magnify-plus"></i> </div>
        <div><i class="fas fa-angle-left"></i></div>
        <div><i class="fas fa-angle-right"></i></div>
      </div>
    </div>
    <div style="display: flex; width: max-content; ">
      <div class="rh-gantt-users-column">
        <div class="rh-gantt-users-column-name">
          Name
        </div>
        <div class="rh-gantt-users-column-row" v-for="row in rowList" :key="row.label">
          <div class="rh-gantt-users-column-number">
            {{ row.label }}
          </div>
          <div style="display: flex;">
            <img src="./assets/img/dan.jpg" alt="" class="rh-gantt-users-column-img">
            <div class="rh-gantt-users-column-fullname"> User Name Last Name</div>
          </div>
        </div>
      </div>
      <div>
        <div class="rf-grantt-timeaxis">
          <div class="rf-grantt-timeaxis-row-0">
            <div class="rf-grantt-timeaxis-cell-0" :style="{ minWidth: widthDefaults + 'px' }" v-for="item in axisDays"
              :key="item.value">
              {{ item.value }}
            </div>
          </div>
          <div style="display: flex; flex-direction: row;">
            <div class="rf-grantt-timeaxis-row-1" v-for="s in axisDays" :key="s.value">
              <div v-for="h  in hours" :key="h" class="rf-grantt-timeaxis-cell-1">{{ h }}</div>
            </div>
          </div>
        </div>
        <div ref="barContainer" class="rh-gantt-bar-container" id="scrollable" @mousemove="getMouseCoordinates">
          <div class="rf-grantt-grid" style="display: flex; flex-direction: row; height: 45px; position: relative;"
            v-for="row in rowList" :key="row.label">
            <div class="rf-grantt-grid-row-1" v-for=" ok in  axisDays" :key="ok.value">
              <div :class="{ 'rf-grantt-grid-hour-cell': n < 9 && n < 18 }" v-for="n in hours " :key="n"
                class="rf-grantt-grid-cell-1"></div>
            </div>
            <g-gantt-bar v-for="bar in row.barList" :key="bar.id" :bar="bar" ref="ganttBar" :bar-start="bar.barStart"
              :bar-end="bar.barEnd" :all-bars-in-row="row.barList" :rectC="rectC" :chartStart="chartStart"
              :chartEnd="chartEnd" :widthAxis="widthAxis" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import GGanttBar from './GGanttBar.vue'
import moment from 'moment'

export default {
  components: {
    GGanttBar
  },

  data() {
    return {
      chartStart: "2020-03-07 00:00",
      chartEnd: "2020-03-15 00:00",

      rectC: {},
      barContainer: {},
      barContainerWidth: 0,
      hours: [...Array(24).keys()],
      widthDefaults: 360,
      hourWidth: 15,
      axisDays: [],
      hourCount: null,
      rowList: [
        {
          label: "1",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "blue", opacity: 0.7, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },

        {
          label: "2",
          barList: [
            {
              myStart: "2020-03-06 09:00",
              myEnd: "2020-03-06 18:00",
              image: "vue_ganttastic_logo_no_text.png",
              label: "I have an image",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2020-03-07 04:00",
              myEnd: "2020-03-07 15:00",
              label: "We belong together ^",
              ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              myStart: "2020-03-08 18:00",
              myEnd: "2020-03-08 22:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#aa34a3" }
            }
          ]
        },

        {
          label: "3",
          barList: [
            {
              myStart: "2020-03-09 09:00",
              myEnd: "2020-03-09 10:00",
              label: "I am with stupid ^",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2020-03-10 22:30",
              myEnd: "2020-03-10 23:00",
              label: "With handles!",
              ganttBarConfig: { color: "white", backgroundColor: "#a23def", handles: true }
            },
            {
              myStart: "2020-03-05 01:00",
              myEnd: "2020-03-05 07:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#5effad", pushOnOverlap: false, zIndex: 3 }
            },
            {
              myStart: "2020-03-07 14:00",
              myEnd: "2020-03-08 20:00",
              label: "Woooow!",
              ganttBarConfig: { color: "white", background: "repeating-linear-gradient(45deg,#de7359,#de7359 10px,#ffc803 10px,#ffc803 20px)" }
            },
          ]
        },

        {
          label: "4",
          barList: [
            {
              myStart: "2020-03-08 06:30",
              myEnd: "2020-03-03 20:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#d18aaf", handles: true }
            },
            {
              myStart: "2020-03-10 00:30",
              myEnd: "2020-03-10 02:00",
              label: "Rectangular",
              ganttBarConfig: { color: "white", backgroundColor: "#f2840f" }
            },
          ]
        },
        {
          label: "5",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "6",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "7",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "8",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "9",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "10",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "11",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "12",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "13",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "14",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },
        {
          label: "15",
          barList: [
            {
              id: '1',
              myStart: "2020-03-05 4:00",
              myEnd: "2020-03-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2020-03-06 18:00",
              myEnd: "2020-03-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2020-03-08 14:00",
              myEnd: "2020-03-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2020-03-09 04:25",
              myEnd: "2020-03-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2020-03-10 04:25",
              myEnd: "2020-03-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },

        {
          label: "16",
          barList: [
            {
              myStart: "2020-03-06 09:00",
              myEnd: "2020-03-06 18:00",
              image: "vue_ganttastic_logo_no_text.png",
              label: "I have an image",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2020-03-07 04:00",
              myEnd: "2020-03-07 15:00",
              label: "We belong together ^",
              ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              myStart: "2020-03-08 18:00",
              myEnd: "2020-03-08 22:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#aa34a3" }
            }
          ]
        },
        {
          label: "17",
          barList: [
            {
              myStart: "2020-03-06 09:00",
              myEnd: "2020-03-06 18:00",
              image: "vue_ganttastic_logo_no_text.png",
              label: "I have an image",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2020-03-07 04:00",
              myEnd: "2020-03-07 15:00",
              label: "We belong together ^",
              ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              myStart: "2020-03-08 18:00",
              myEnd: "2020-03-08 22:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#aa34a3" }
            }
          ]
        },
      ]
    }
  },
  computed: {
    widthAxis() {
      return this.getHourCount() * this.hourWidth
    }
  },
  mounted() {

    this.initAxisDaysAndHours()
    this.barContainer = this.$refs.barContainer
    const barContainer = this.barContainer
    this.rectC = barContainer.getBoundingClientRect();
    document.addEventListener('scroll', this.handleScroll);
  },
  updated() {

  },
  beforeDestroy() {
    // Ensure to remove the scroll event listener when the component is destroyed
    document.removeEventListener('scroll', this.handleScroll);
  },

  methods: {
    handleScroll(event) {
      console.log("Scroll position:", event.target.scrollTop);
    },
    getHourCount() {
      let momentChartStart = moment(this.chartStart, 'YYYY-MM-DD HH:mm')
      let momentChartEnd = moment(this.chartEnd, 'YYYY-MM-DD HH:mm')

      return Math.floor(momentChartEnd.diff(momentChartStart, "hour", true))
    },
    getMouseCoordinates() {
      if (this.$refs.barContainer) {
        const barContainer = this.$refs.barContainer;
        this.rectC = barContainer.getBoundingClientRect();
        // Координаты мыши относительно верхнего левого угла прокручиваемого div
        // const mouseX = event.clientX - this.rectC.left;
        // const mouseY = event.clientY - this.rectC.top;
        // console.log('Координаты мыши внутри div:', this.rectC, { x: mouseX, y: mouseY }, event.clientX, event.clientY);
      }
    },

    initAxisDaysAndHours() {
      this.axisDays = []
      let start = moment(this.chartStart, 'YYYY-MM-DD HH:mm')
      let end = moment(this.chartEnd, 'YYYY-MM-DD HH:mm')
      this.hourCount = Math.floor(end.diff(start, "hour", true))
      while (start.isBefore(end)) {
        let hourCountOfDay = start.format("DD.MM.YYYY") == end.format("DD.MM.YYYY") ? end.hour() : 24 - start.hour()
        let widthPercentage = hourCountOfDay / this.hourCount * 100
        let endHour = start.day() === end.day() ? end.hour() - 1 : 23
        // -1 because the last hour is not included e.g if chartEnd=04:00 the last interval we display is between 03 and 04
        this.axisDays.push(this.getAxisDayObject(start, widthPercentage, endHour))
        start.add(1, "day").hour(0)
      }
    },
    getAxisDayObject(datetime, widthPercentage, endHour) {
      let datetimeMoment = moment(datetime, 'YYYY-MM-DD HH:mm')
      let axisDayObject = {
        widthPercentage: widthPercentage,
        value: datetime.format("YYYY-MM-DD"),
        ganttHours: []
      }
      let startHour = datetimeMoment.hour()
      for (let i = 0; i <= (endHour - startHour); i++) {
        let hour = {
          text: datetimeMoment.format("HH"),
          fullDatetime: datetimeMoment.format("DD.MM.YYYY HH:mm")
        }
        axisDayObject.ganttHours.push(hour)
        datetimeMoment.add(1, "hour")
      }
      return axisDayObject
    },
  }
}
</script>

<style>
#scrollabe {
  overflow-y: auto;
}

.rh-gantt-bar-container {
  margin-top: 104px;
}

.rh-gantt-users-column-fullname {
  font-weight: 500;
  font-family: sans-serif;
  margin-left: 7px;
  display: flex;
  justify-content: start;
  align-items: center;
  font-size: 14px;
  width: 180px;
  color: rgb(102, 114, 128);
}

.rh-gantt-users-column-img {
  min-width: 37px;
  max-width: 37px;
  height: 37px;
  border-radius: 30px;
  margin: 4px;
  object-fit: cover;
  border: 1px solid #e9eaeb;
  padding: 2px;
}

.rh-gantt-users-column-name {
  display: flex;
  position: sticky;
  top: 104px;
  z-index: 9999;
  background: rgb(243, 244, 245);
  height: 58px;
  box-shadow: rgba(0, 0, 0, 0.08) 0px 2px 4px;
  border-bottom: 1px solid #e9eaeb;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  color: #667280;
  font-size: 14px;
  font-family: sans-serif;
}

.rh-gantt-users-column-row {
  border-bottom: 1px solid #e9eaeb;
  height: 45px;
  display: flex;
  flex-direction: row;
}

.rh-gantt-users-column-number {
  border-inline: 1px solid rgb(233, 234, 235);
  width: 60px;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  color: rgb(102, 114, 128);
  font-size: 14px;
  font-weight: 500;
  color: #667280;
  font-family: sans-serif;
}

.rh-gantt-users-column {
  position: sticky;
  z-index: 99990;
  left: 0;
  background: white;
  min-width: 260px;
  border-right: 7px solid #e5e5e8;
  margin-top: 104px;
}

.rh-gantt-top-bar {
  display: flex;
  justify-content: start;
  position: fixed;
  left: 0px;
  z-index: 99999;
  top: 52px;
  min-width: 100vw;
  background: rgb(243, 244, 245);
  border-bottom: 1px solid #e5e5e8;
  height: 52px;
}

.rf-grantt-timeaxis {
  position: sticky;
  box-shadow: 0 2px 4px rgba(0, 0, 0, .0784313725490196);
  top: 104px;
  z-index: 9999;
}

.rf-grantt-timeaxis-row-0 {
  display: flex;
  flex-direction: row;
  border-bottom: 1px solid #d8d9da;
  width: max-content;
  background: #f3f4f5;
}

.rf-grantt-timeaxis-cell-0 {
  /* background: grey; */
  text-align: center;
  border-inline-end: 1px solid #d8d9da;
  height: 28px;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  color: #667280;
  font-family: sans-serif;
  font-size: 14px;
  font-weight: 500;
}

.rf-grantt-timeaxis-row-1 {
  display: flex;
  flex-direction: row;
  border-bottom: 1px solid #d8d9da;
  background: #f3f4f5;
}

.rf-grantt-timeaxis-cell-1 {
  /* background: grey; */
  font-size: 9px;
  width: 15px;
  text-align: center;
  border-inline-end: 1px solid #d8d9da;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 500;
  color: #667280;
  font-family: sans-serif;
}

.rf-grantt-grid-row-1 {
  display: flex;
  flex-direction: row;
  border-bottom: 1px solid #e9eaeb;
  background: #f3f4f5;
}

.rf-grantt-grid-cell-1 {
  font-size: 12px;
  width: 15px;
  text-align: center;
  border-inline-end: 1px solid #e9eaeb;
  background: #ffffff;
}

.rf-grantt-grid-hour-cell {
  background: #f7f7f7;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>