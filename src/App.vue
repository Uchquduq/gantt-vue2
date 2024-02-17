<template>
  <div style="position: relative; overflow: hidden;">
    <!-- {{users}} -->
    <div class="rh-gantt-top-bar">
      <div class="rh-gantt-top-bar-buttons">
        <div @click="zoomPlus"><i class="fa-solid fa-magnifying-glass-plus rh-gantt-top-bar-btn"></i> </div>
        <div @click="zoomMinus"><i class="fa-solid fa-magnifying-glass-minus rh-gantt-top-bar-btn"></i></div>
        <div @click="scrollRight"><i class="fas fa-angle-left rh-gantt-top-bar-btn"></i></div>
        <div @click="scrollLeft"><i class="fas fa-angle-right rh-gantt-top-bar-btn"></i></div>
      </div>
      <div class="rh-gantt-top-fullscreen">
        <button class="rh-gantt-top-bar-btn" v-if="!fullscreen" @click="toggleFullScreen">
          <i class="fa-solid fa-expand"></i>
        </button>
        <button class="rh-gantt-top-bar-btn" v-if="fullscreen" @click="exitFullScreen">
          <i class="fa-solid fa-compress"></i>
        </button>
      </div>
      <div class="rh-gantt-top-bar-buttons">
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #00AB55;"></i>NEW</div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #FFC107;"></i>IN PROCESS
        </div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #f18a2d;"></i>DONE</div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #0177c0;"></i>ESTIMATED
        </div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #7e349d;"></i>CLOSED</div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #e74c3c;"></i>CANCELED
        </div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #08aba0;"></i>ON HOLD</div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #08aba0;"></i>REJECTED TO
          OWNER</div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #08aba0;"></i>REJECTED TO
          EXECUTOR</div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #08aba0;"></i>BACKLOG</div>
        <div class="rh-gantt-task-type"><i class="fa-solid fa-circle" style="color: #08aba0;"></i>REJECTED
        </div>
      </div>
      <!--       <div class="rh-gantt-datepicker">
       <input type="date" id="start" name="trip-start" value="2024-02-12" min="2023-01-01" />
      </div> -->
    </div>
    <div v-if="render" style="display: flex; width: max-content;     overflow: scroll;
    display: flex;
    width: 95vw;
    height: 90vh; " ref="gantBigContainer">
      <div class="rh-gantt-users-column">
        <div class="rh-gantt-users-column-name">
          Name
        </div>
        <div class="rh-gantt-users-column-row" v-for="(row, index) in rowList" :key="row.label">
          <div class="rh-gantt-users-column-number">
            {{ index + 1 }}
          </div>
          <div style="display: flex;">
            <!-- <img v-if="row.path" :src="baseUrl + row.path" alt="" class="rh-gantt-users-column-img"> -->
            <img src="./assets/img/rufat.png" class="rh-gantt-users-column-img">
            <div class="rh-gantt-users-column-fullname">
              {{ row.name }}
            </div>
          </div>
        </div>
      </div>
      <div style="height: max-content;">
        <div class="rf-grantt-timeaxis">
          <div class="rf-grantt-timeaxis-row-0">
            <div class="rf-grantt-timeaxis-cell-0"
              :style="{ minWidth: ((zoom == 'month' || zoom == 'week') ? widthHourAxis * item.ganttHours.length : widthDefaults) + 'px' }"
              v-for="item in axisDays" :key="item.value">
              <template v-if="zoom == 'month'">
                {{ item.value }}
              </template>
              <template v-if="zoom == 'week'">
                {{ item.value }}
              </template>
              <template v-if="zoom == 'day'">
                {{ item.value }}
              </template>
            </div>
          </div>
          <div style="display: flex; flex-direction: row;">
            <div class="rf-grantt-timeaxis-row-1" v-for="s in axisDays" :key="s.value">
              <div v-for="h in s.ganttHours" :key="h.fullDatetime" :style="{ width: widthHourAxis + 'px' }"
                class="rf-grantt-timeaxis-cell-1">
                {{ h.text }}
              </div>
            </div>
          </div>
        </div>
        <div ref="barContainer" class="rh-gantt-bar-container" id="scrollable" @mousemove="getMouseCoordinates"
          style="padding-bottom: 20px;">
          <div class="rf-grantt-grid" style="display: flex; flex-direction: row; height: 45px; position: relative;"
            v-for="row in rowList" :key="row.label">
            <div class="rf-grantt-grid-row-1" v-for=" ok in  axisDays" :key="ok.value">
              <div :style="{ width: widthHourAxis + 'px' }"
                :class="{ 'rf-grantt-grid-hour-cell': (n < 9 && n < 18) && zoom == 'hours' }" v-for="n in ok.ganttHours"
                :key="n.fullDatetime" class="rf-grantt-grid-cell-1" @dblclick="createTaskModal(n.fullDatetime, row.id)">

              </div>
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
import data from '../data'
import moment from 'moment'
import axios from 'axios'

export default {
  components: {
    GGanttBar
  },
  data() {
    return {
      fullscreen: false,
      baseUrl: '/uploads/images/',
      oldStart: moment().set({ hour: 0, minute: 0, second: 0, millisecond: 0 }),
      oldEnd: moment().set({ hour: 0, minute: 0, second: 0, millisecond: 0 }),
      chartStart: moment().set({ hour: 0, minute: 0, second: 0, millisecond: 0 }).subtract(2, 'week').format('YYYY-MM-DD HH:mm'),
      chartEnd: moment().set({ hour: 0, minute: 0, second: 0, millisecond: 0 }).add(2, 'month').format('YYYY-MM-DD HH:mm'),
      zoom: 'week',
      zoomInt: 1,
      render: true,
      rectC: {},
      barContainer: {},
      barContainerWidth: 0,
      hours: [...Array(24).keys()],
      widthDefaults: 360,
      hourWidth: 20,
      axisDays: [],
      hourCount: null,
      rowList: [
        {
          label: "1",
          barList: [
            {
              id: '1',
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "blue", opacity: 0.7, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },

        {
          label: "2",
          barList: [
            {
              myStart: "2024-02-06 09:00",
              myEnd: "2024-02-06 18:00",
              image: "vue_ganttastic_logo_no_text.png",
              label: "I have an image",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2024-02-07 04:00",
              myEnd: "2024-02-07 15:00",
              label: "We belong together ^",
              ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              myStart: "2024-02-08 18:00",
              myEnd: "2024-02-08 22:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#aa34a3" }
            }
          ]
        },

        {
          label: "3",
          barList: [
            {
              myStart: "2024-02-09 09:00",
              myEnd: "2024-02-09 10:00",
              label: "I am with stupid ^",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2024-02-10 22:30",
              myEnd: "2024-02-10 23:00",
              label: "With handles!",
              ganttBarConfig: { color: "white", backgroundColor: "#a23def", handles: true }
            },
            {
              myStart: "2024-02-05 01:00",
              myEnd: "2024-02-05 07:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#5effad", pushOnOverlap: false, zIndex: 3 }
            },
            {
              myStart: "2024-02-07 14:00",
              myEnd: "2024-02-08 20:00",
              label: "Woooow!",
              ganttBarConfig: { color: "white", background: "repeating-linear-gradient(45deg,#de7359,#de7359 10px,#ffc803 10px,#ffc803 20px)" }
            },
          ]
        },

        {
          label: "4",
          barList: [
            {
              myStart: "2024-02-08 06:30",
              myEnd: "2024-02-09 20:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#d18aaf", handles: true }
            },
            {
              myStart: "2024-02-10 00:30",
              myEnd: "2024-02-10 02:00",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
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
              myStart: "2024-02-05 4:00",
              myEnd: "2024-02-06 6:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "green", opacity: 0.5, immobile: true }
            },
            {
              id: '2',
              myStart: "2024-02-06 18:00",
              myEnd: "2024-02-07 23:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '3',
              myStart: "2024-02-08 14:00",
              myEnd: "2024-02-08 16:00",
              label: "Immobile",
              // ganttBarConfig: { color: "white", backgroundColor: "#404040", opacity: 0.5, immobile: true }
            },
            {
              id: '4',
              myStart: "2024-02-09 04:25",
              myEnd: "2024-02-09 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              id: '5',
              myStart: "2024-02-10 04:25",
              myEnd: "2024-02-10 10:30",
              label: "Bar",
              // ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            }
          ]
        },

        {
          label: "16",
          barList: [
            {
              myStart: "2024-02-06 09:00",
              myEnd: "2024-02-06 18:00",
              image: "vue_ganttastic_logo_no_text.png",
              label: "I have an image",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2024-02-07 04:00",
              myEnd: "2024-02-07 15:00",
              label: "We belong together ^",
              ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              myStart: "2024-02-08 18:00",
              myEnd: "2024-02-08 22:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#aa34a3" }
            }
          ]
        },
        {
          label: "17",
          barList: [
            {
              myStart: "2024-02-06 09:00",
              myEnd: "2024-02-06 18:00",
              image: "vue_ganttastic_logo_no_text.png",
              label: "I have an image",
              ganttBarConfig: { color: "white", backgroundColor: "#de3b26", bundle: "redBundle" }
            },
            {
              myStart: "2024-02-07 04:00",
              myEnd: "2024-02-07 15:00",
              label: "We belong together ^",
              ganttBarConfig: { color: "white", backgroundColor: "#2e74a3", bundle: "blueBundle" }
            },
            {
              myStart: "2024-02-08 18:00",
              myEnd: "2024-02-08 22:00",
              label: "Bar",
              ganttBarConfig: { color: "white", backgroundColor: "#aa34a3" }
            }
          ]
        },
      ],
      projectData: [],
      tasks: [],
      users: []
    }
  },
  computed: {
    widthAxis() {
      return this.getHourCount() * this.widthHour
    },
    widthHour() {
      return this.zoom === 'day' ? 15 : (this.zoom === 'month' ? 20 / 24 : (this.zoom === 'week' ? 20 / 24 : 20 / 24));
    },
    widthHourAxis() {
      return this.zoom === 'day' ? 15 : (this.zoom === 'month' ? 20 : (this.zoom === 'week' ? 20 : 15));
    }
  },
  created() {
    this.updateFullScreenState();
    document.addEventListener("fullscreenchange", this.updateFullScreenState);
  },
  destroyed() {
    document.removeEventListener("fullscreenchange", this.updateFullScreenState);
  },
  mounted() {
    this.initAxisDaysAndHours()
    this.barContainer = this.$refs.barContainer
    // this.fetchData()
    this.fetchLocalData()
    const barContainer = this.barContainer
    this.rectC = barContainer.getBoundingClientRect()
  },
  updated() {

  },
  methods: {
    createTaskModal(hour, executorId) {
      console.log(hour, executorId)
      if (hour && executorId) {
        let url = `forms/tasks?_target=modal_1200&startTime=${hour}&executorId=${executorId}"`
        this.$root.navigateTo(url)
      }
    },
    // --- fullscreen btn --------------------------
    toggleFullScreen() {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen();
      }
      else {
        document.exitFullscreen();
      }
    },
    exitFullScreen() {
      document.exitFullscreen();
    },
    updateFullScreenState() {
      this.fullscreen = Boolean(document.fullscreenElement);
    },
    // --- fetch local data ------------------------
    fetchLocalData() {
      this.projectData = data
      this.users = data.members
      this.tasks = data.project_task
      this.users.forEach(u => {
        u.barList = this.tasks.filter(t => t.executor_id == u.id)
      })
      this.users.forEach(u => {
        u.label = u.id
        u.barList = this.tasks.filter(t => t.executor_id == u.id)
          .map(function (i) {
            return {
              'id': i.id,
              'myStart': moment(i.task_date, 'DD.MM.YYYY HH:mm').format('YYYY-MM-DD HH:mm'),
              'myEnd': moment(i.term_date, 'DD.MM.YYYY HH:mm').format('YYYY-MM-DD HH:mm'),
              'label': i.name,
              'task_state': i.task_state
            }
          })
      })
      this.rowList = this.users
      console.log(data)
    },
    // --- fetch api data --------------------------
    fetchData() {
      axios.get(`../data.json`).then(res => {
        this.projectData = res.data.data
        this.users = res.data.data.members
        this.tasks = res.data.data.project_task
        this.users.forEach(u => {
          u.barList = this.tasks.filter(t => t.executor_id == u.id)
        })
        this.users.forEach(u => {
          u.label = u.id
          u.barList = this.tasks.filter(t => t.executor_id == u.id)
            .map(function (i) {
              return {
                'id': i.id,
                'myStart': moment(i.task_date, 'DD.MM.YYYY HH:mm').format('YYYY-MM-DD HH:mm'),
                'myEnd': moment(i.term_date, 'DD.MM.YYYY HH:mm').format('YYYY-MM-DD HH:mm'),
                'label': i.name,
                'task_state': i.task_state
              }
            })
        })
        this.rowList = this.users
        console.log(this.users)
      })
    },
    scrollRight() {
      // Убедимся, что элемент правильно получен
      var element = this.$refs.gantBigContainer;
      console.log("Scroll Left:", element.scrollLeft);
      console.log("Scroll Width:", element.scrollWidth, "Offset Width:", element.offsetWidth);
      if (!element) {
        console.error("Element with ref 'barContainer' not found.");
        return;
      }
      // Убедимся, что элемент - контейнер, который можно прокручивать горизонтально
      if (element.offsetWidth >= element.scrollWidth) {
        console.warn("Element is not horizontally scrollable.");
        return;
      }
      // Увеличим значение scrollLeft на 20 пикселей
      element.scrollLeft -= element.scrollWidth / 16;
      // Выведем в консоль текущую позицию прокрутки и размеры элемента
    },
    scrollLeft() {
      // Убедимся, что элемент правильно получен
      var element = this.$refs.gantBigContainer;
      console.log("Scroll Left:", element.scrollRight);
      console.log("Scroll Width:", element.scrollWidth, "Offset Width:", element.offsetWidth);
      if (!element) {
        console.error("Element with ref 'barContainer' not found.");
        return;
      }
      // Убедимся, что элемент - контейнер, который можно прокручивать горизонтально
      if (element.offsetWidth >= element.scrollWidth) {
        console.warn("Element is not horizontally scrollable.");
        return;
      }
      // Увеличим значение scrollLeft на 20 пикселей
      element.scrollLeft += element.scrollWidth / 16;

      // Выведем в консоль текущую позицию прокрутки и размеры элемента
    },
    zoomPlus() {
      if (this.zoomInt <= 2 && this.zoomInt > 0) {
        this.render = false
        this.zoomInt = this.zoomInt - 1
        if (this.zoomInt == 1) {
          this.zoom = 'week'
          this.hourWidth = 20
          this.initAxisDaysAndHours()
        } else if (this.zoomInt == 2) {
          this.zoom = 'month'
          this.hourWidth = 20
          this.initAxisDaysAndHours()
        } else if (this.zoomInt == 0) {
          this.zoom = 'day'
          this.hourWidth = 15
          this.initAxisDaysAndHours()
        }
        this.render = true
      }
    },
    zoomMinus() {
      if (this.zoomInt < 2 && this.zoomInt >= 0) {
        this.render = false
        this.zoomInt = this.zoomInt + 1
        if (this.zoomInt == 1) {
          this.zoom = 'week'
          this.hourWidth = 20
          this.initAxisDaysAndHours()
        } else if (this.zoomInt == 2) {
          this.zoom = 'month'
          this.hourWidth = 20
          this.initAxisDaysAndHours()
        }
        this.render = true
      }
    },
    getHourCount() {
      let momentChartStart = moment(this.chartStart, 'YYYY-MM-DD HH:mm').set({ hour: 0, minute: 0, second: 0, millisecond: 0 })
      let momentChartEnd = moment(this.chartEnd, 'YYYY-MM-DD HH:mm').set({ hour: 0, minute: 0, second: 0, millisecond: 0 })
      console.log(Math.floor(momentChartEnd.diff(momentChartStart, "hour", true)))
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
      if (this.zoom == 'day') {
        this.chartStart = moment(this.oldStart, 'YYYY-MM-DD HH:mm').set({ hour: 0, minute: 0, second: 0, millisecond: 0 }).subtract(2, 'week').subtract(1, 'day')
        this.chartEnd = moment(this.oldStart, 'YYYY-MM-DD HH:mm').set({ hour: 0, minute: 0, second: 0, millisecond: 0 }).add(2, 'week').subtract(4, 'day')
        let start = moment(this.oldStart, 'YYYY-MM-DD HH:mm').set({ hour: 0, minute: 0, second: 0, millisecond: 0 }).subtract(2, 'week').subtract(1, 'day')
        let end = moment(this.oldStart, 'YYYY-MM-DD HH:mm').set({ hour: 0, minute: 0, second: 0, millisecond: 0 }).add(2, 'week').subtract(4, 'day')
        this.hourCount = Math.floor(end.diff(start, "hour", true))
        while (start.isBefore(end)) {
          let hourCountOfDay = start.format("DD.MM.YYYY") == end.format("DD.MM.YYYY") ? end.hour() : 24 - start.hour()
          let widthPercentage = hourCountOfDay / this.hourCount * 100
          let endHour = start.day() === end.day() ? end.hour() - 1 : 23
          // -1 because the last hour is not included e.g if chartEnd=04:00 the last interval we display is between 03 and 04
          this.axisDays.push(this.getAxisDayObject(start, widthPercentage, endHour))
          start.add(1, "day").hour(0)
        }
        console.log(this.axisDays)
      } else if (this.zoom == 'month') {
        const result = [];
        let currentDate = moment(this.oldStart, 'YYYY-MM-DD HH:mm').subtract(1, 'month').clone();
        while (currentDate.isSameOrBefore(this.chartEnd, 'YYYY-MM-DD HH:mm')) {
          const month = currentDate.format('DD-MM-YYYY');
          const daysInMonth = [];
          const daysCount = currentDate.daysInMonth();
          // Populate the array with each day of the month
          for (let i = 1; i <= daysCount; i++) {
            const day = currentDate.clone().date(i).format('DD-MM-YYYY');
            daysInMonth.push({ fullDatetime: day, text: moment(day, 'DD.MM.YYYY').format('DD') });
          }
          // Add month object to the result array
          result.push({ value: moment(month, 'DD-MM-YYYY').format('MMM YYYY'), ganttHours: daysInMonth });
          // Move to the next month

          currentDate.add(1, 'month');
        }
        this.axisDays = result
        console.log(this.axisDays)
        this.chartStart = moment(result[0].ganttHours[0].fullDatetime, 'DD-MM-YYYY').format('YYYY-MM-DD HH:mm')
        console.log(result[0].ganttHours[0].fullDatetime)
        this.chartEnd = moment(result[result.length - 1].ganttHours[result[result.length - 1].ganttHours.length - 1].fullDatetime, 'DD-MM-YYYY').format('YYYY-MM-DD HH:mm')
        console.log(result[result.length - 1].ganttHours[result[result.length - 1].ganttHours.length - 1].fullDatetime)
        this.getHourCount()
      } else if (this.zoom == 'week') {
        const result = [];
        let currentDate = moment(this.oldStart, 'YYYY-MM-DD HH:mm').subtract(4, 'week').startOf('week');
        const endDate = moment(this.oldStart, 'YYYY-MM-DD HH:mm').add(10, 'week');

        while (currentDate.isBefore(endDate)) {
          const weekStart = currentDate.format('DD-MM-YYYY');
          // const weekEnd = currentDate.clone().endOf('week').format('DD-MM-YYYY');
          const daysInWeek = [];

          for (let i = 0; i < 7; i++) {
            const day = currentDate.clone().add(i, 'day').format('DD-MM-YYYY');
            // if (moment(day, 'DD-MM-YYYY').isSameOrBefore(endDate)) {
            daysInWeek.push({ fullDatetime: day, text: moment(day, 'DD-MM-YYYY').format('dd') });
            // } else {
            //   break;
            // }
          }

          result.push({ value: weekStart, ganttHours: daysInWeek });
          currentDate.add(1, 'week');
        }
        this.axisDays = result
        console.log(this.axisDays)
        this.chartStart = moment(result[0].ganttHours[0].fullDatetime, 'DD-MM-YYYY').format('YYYY-MM-DD HH:mm')
        console.log(result[0].ganttHours[0].fullDatetime)
        this.chartEnd = moment(result[result.length - 1].ganttHours[result[result.length - 1].ganttHours.length - 1].fullDatetime, 'DD-MM-YYYY').format('YYYY-MM-DD HH:mm')
        console.log(result[result.length - 1].ganttHours[result[result.length - 1].ganttHours.length - 1].fullDatetime)
        this.getHourCount()
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
  margin-top: 52px;
}

.rh-gantt-top-fullscreen {
  margin: 6px;
}

.rh-gantt-top-bar-buttons {
  display: flex;
  font-family: sans-serif;
  flex-direction: row;
  justify-content: start;
  margin: 6px;
}

.rh-gantt-top-bar-btn {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 36px;
  width: 72px;
  border: 1px solid #e9eaeb;
  background: #edeef0;
  color: #667280;
  font-size: 14px;
  cursor: pointer;
  transition: all .3s ease;
}

.rh-gantt-task-type {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 4px;
  height: 36px;
  padding: 8px;
  width: auto;
  border: 1px solid #e9eaeb;
  background: #edeef0;
  color: #667280;
  font-size: 10px;
  transition: all .3s ease;
  border-inline-end: 1px solid #e9eaeb;
}

.rh-gantt-task-type i {
  font-size: 10px;
}

.rh-gantt-top-bar-btn:hover {
  background: #e1e1e1;
  color: #3f4a57;
}

.rh-gantt-users-column-fullname {
  margin-right: 5px;
  margin-top: 13px;
  font-weight: 500;
  font-family: sans-serif;
  margin-left: 7px;
  display: block;
  font-size: 14px;
  width: 180px;
  color: rgb(102, 114, 128);
  line-height: 1.3;
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
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
  top: 52px;
  z-index: 9999;
  background: rgb(243, 244, 245);
  height: 58px;
  box-shadow: rgba(0, 0, 0, 0.08) 0px 2px 4px;
  border-bottom: 1px solid #e9eaeb;
  border-inline-start: 1px solid #e9eaeb;
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
  min-width: 60px;
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
  min-width: 310px;
  border-right: 7px solid #e5e5e8;
  border-left: 7px solid #f3f4f5;
  margin-top: 52px;
  height: max-content;
}

.rh-gantt-top-bar {
  display: flex;
  justify-content: start;
  position: fixed;
  /*   left: 0px; */
  z-index: 99999;
  /*   top: 52px; */
  min-width: 100vw;
  background: rgb(243, 244, 245);
  border-bottom: 1px solid #e5e5e8;
  margin-left: 7px;
  height: 52px;
}

.rf-grantt-timeaxis {
  position: sticky;
  box-shadow: 0 2px 4px rgba(0, 0, 0, .0784313725490196);
  top: 52px;
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