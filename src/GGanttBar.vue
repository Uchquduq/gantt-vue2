<template>
  <div>
    <!-- @mouseenter.stop="onMouseenter($event)"
      @mouseleave.stop="onMouseleave($event)" @mousedown.stop="onMousedown($event)" @click.stop="onClick($event)"
      @dblclick="onDblclick($event)" @contextmenu="onContextmenu($event)" -->
    {{ mouseOver }}
    <div class="g-gantt-bar" ref="g-gantt-bar" style="cursor: pointer; background: #8ee997;" :style="barStyle">

      <div class="g-gantt-bar-label" @mousedown.stop="startDrag($event, bar, 'drag')">
        <span style="">
          <!-- {{ bar.myStart | TimeFilter }} - {{ bar.myEnd | TimeFilter }} --> Task
        </span>
        <slot name="bar-label" :bar="bar">
          {{ barConfig.label || "" }}
        </slot>
      </div>
      <!-- v-if="barConfig.handles" -->
      <template>
        <div @mousedown.stop="startDrag($event, bar, 'start')" class="g-gantt-bar-handle-left"></div>
        <div @mousedown.stop="startDrag($event, bar, 'end')" class="g-gantt-bar-handle-right"></div>
      </template>
    </div>

    <transition name="fade" mode="out-in">
      <div style="white-space: nowrap;" v-if="!barConfig.noTooltip && (showTooltip || isDragging)" class="g-gantt-tooltip"
        :style="tooltipStyle">
        <div class="color-indicator" :style="{ background: this.barStyle.background || this.barStyle.backgroundColor }">
        </div>
        {{ bar.myStart | TimeFilter }} - {{ bar.myEnd | TimeFilter }}

      </div>
    </transition>

  </div>
</template>

<script>
import moment from 'moment'

export default {
  name: "GGanttBarNew",

  props: {
    widthAxis: { type: Number },
    chartStart: { type: String },
    chartEnd: { type: String },
    rectC: { type: [Object, DOMRect] },
    // barContainerWidth: { type: Number },
    bar: { type: Object },
    barStart: { type: String }, // property name of the bar objects that represents the start datetime
    barEnd: { type: String }, // property name of the bar objects that represents the end datetime,
    allBarsInRow: { type: Array },
  },

  data() {
    return {
      mouseOver: '',

      showTooltip: false,
      tooltipTimeout: null,
      dragLimitLeft: null,
      dragLimitRight: null,
      isDragging: false,
      isMainBarOfDrag: false, // is this the bar that was clicked on when starting to drag
      // or is it dragged along some other bar from the same bundle
      cursorOffsetX: 0,
      mousemoveCallback: null,  // gets initialized when starting to drag
      // possible values: drag, dragByHandleLeft, dragByHandleRight,
      barStartBeforeDrag: null,
      barEndBeforeDrag: null,



      // unit: 'month', // Initial zoom level
      // totalDays: 30, // Number of days in a month
      // totalMonths: 12, // Number of months in a year
      draggedTask: null,
      dragEdge: null,
      // initialX: 0,
      // initialStart: 0,
      // initialDuration: 0,

      initialX1: 0,
      initialX2: 0,
      dragX1: 0,
      dragX2: 0
    }
  },

  computed: {

    // use these computed moment objects to work with the bar's start/end dates:
    // instead of directly mutating them:
    // barStartMoment: {
    //   get() {
    //     return moment(this.bar[this.barStart])
    //   },
    //   /* eslint-disable */
    //   set(value) {
    //     this.bar[this.barStart] = moment(value).format("YYYY-MM-DD HH:mm:ss")
    //   }
    // },

    // barEndMoment: {
    //   get() {
    //     return moment(this.bar[this.barEnd])
    //   }
    //   ,
    //   set(value) {
    //     this.bar[this.barEnd] = moment(value).format("YYYY-MM-DD HH:mm:ss")
    //   }
    // },

    barConfig() {
      if (this.bar.ganttBarConfig) {
        return {
          ...this.bar.ganttBarConfig,
          background: this.bar.ganttBarConfig.isShadow ? "grey" : this.bar.ganttBarConfig.background || this.bar.ganttBarConfig.backgroundColor,
          opacity: this.bar.ganttBarConfig.isShadow ? "0.3" : this.bar.ganttBarConfig.opacity
        }
      }
      return {}
    },

    barStyle() {
      let xStart = this.mapTimeToPosition(this.bar.myStart)
      let xEnd = this.mapTimeToPosition(this.bar.myEnd)
      return {
        ...(this.barConfig || {}),
        left: `${xStart}px`,
        width: `${xEnd - xStart}px`,
        // height: `${this.ganttChartProps.rowHeight - 6}px`,
        zIndex: this.barConfig.zIndex || (this.isDragging ? 2 : 1)
      }
    },

    tooltipStyle() {
      return {
        left: this.barStyle.left,
        // top: `${this.ganttChartProps.rowHeight}px`,
      }
    },

    chartStartMoment() {
      return moment(this.chartStart)
    },

    chartEndMoment() {
      return moment(this.chartEnd)
    }
  },

  methods: {
    startDrag(event, task, edge) {
      this.isDragging = true;
      this.draggedTask = task;
      this.dragEdge = edge;

      this.initialX1 = this.mapTimeToPosition(this.bar.myStart)
      this.initialX2 = this.mapTimeToPosition(this.bar.myEnd)
      this.dragX1 = event.clientX - this.rectC.left

      // this.initialStart = task.myStart;

      document.addEventListener('mousemove', this.handleDrag);
      document.addEventListener('mouseup', this.stopDrag);
    },
    handleDrag(event) {
      if (this.isDragging) {
        // const deltaX = event.clientX - this.initialX;
        this.dragX2 = (event.clientX - this.rectC.left) - this.dragX1
        // console.log(event.clientX, this.dragX1, this.dragX2)
        if (this.dragEdge === 'start') {
          console.log('start')
          // console.log('start', event)
          this.draggedTask.myStart = this.mapPositionToTime(event.clientX - this.rectC.left)
          // console.log(this.mapPositionToTime(event.clientX))
        } else if (this.dragEdge === 'end') {
          console.log('end')
          this.draggedTask.myEnd = this.mapPositionToTime(event.clientX - this.rectC.left)
          // this.draggedTask.duration = Math.max(1, this.initialDuration + deltaX / 30);
        } else if (this.dragEdge === 'drag') {
          console.log('drag')
          // let startTask = this.mapTimeToPosition(this.draggedTask.myStart)
          // console.log(startTask)
          this.draggedTask.myStart = this.mapPositionToTime(this.initialX1 + this.dragX2)
          this.draggedTask.myEnd = this.mapPositionToTime(this.initialX2 + this.dragX2)
          // this.draggedTask.myStart = this.mapPositionToTime(event.clientX - this.rectC.left)

        }
      }
    },
    stopDrag() {
      this.isDragging = false;
      this.draggedTask = null;
      this.dragEdge = null;
      this.dragX1 = 0
      this.dragX2 = 0
      this.initialX1 = 0
      this.initialX2 = 0
      document.removeEventListener('mousemove', this.handleDrag);
    },
    getHourCount() {
      let momentChartStart = moment(this.chartStart, 'YYYY-MM-DD HH:mm')
      let momentChartEnd = moment(this.chartEnd, 'YYYY-MM-DD HH:mm')
      return Math.floor(momentChartEnd.diff(momentChartStart, "hour", true))
    },
    /* --------------------------------------------------------- */
    /* ------- MAPPING POSITION TO TIME (AND VICE VERSA) ------- */
    /* --------------------------------------------------------- */
    mapTimeToPosition(time) {
      let hourDiffFromStart = moment(time, 'YYYY-MM-DD HH:mm').diff(this.chartStartMoment, "hour", true)
      // console.log('hourd', hourDiffFromStart)
      // console.log('gh', this.getHourCount())
      // console.log(this.rectC.width)
      return (hourDiffFromStart / this.getHourCount()) * this.widthAxis
    },
    mapPositionToTime(xPos) {
      let hourDiffFromStart = (xPos / this.widthAxis) * this.getHourCount()
      return this.chartStartMoment.clone().add(hourDiffFromStart, "hours")
    },
    // onMousedown(e) {
    //   e.preventDefault()
    //   if (e.button === 2) {
    //     return
    //   }
    //   if (!this.barConfig.immobile && !this.barConfig.isShadow) {
    //     this.setDragLimitsOfGanttBar(this)
    //     window.addEventListener("mousemove", this.onFirstMousemove, { once: true })
    //     window.addEventListener("mouseup",
    //       () => window.removeEventListener("mousemove", this.onFirstMousemove),
    //       { once: true }
    //     )
    //   }
    //   const time = this.mapPositionToTime(e.clientX - this.barContainer.left).format("YYYY-MM-DD HH:mm:ss")
    //   this.onBarEvent({ event: e, type: e.type, time }, this)
    // },
  },

  filters: {
    TimeFilter(value) {
      return moment(value, 'YYYY-MM-DD HH:mm').format("DD-MM-YYYY HH:mm")
    }
  },


}
</script>

<style scoped>
.g-gantt-bar {
  position: absolute;
  top: 8px;
  left: 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
  width: 300px;
  height: 28px;
  border-radius: 4px;
  background: #79869c;
  overflow: hidden;
}

.g-gantt-bar-label {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding: 0 14px 0 14px;
  /* 14px is the width of the handle */
  display: flex;
  justify-content: center;
  align-items: center;
}

.g-gantt-bar-label>* {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.g-gantt-bar>.g-gantt-bar-handle-left,
.g-gantt-bar>.g-gantt-bar-handle-right {
  position: absolute;
  width: 3px;
  height: 100%;
  background: #e2ffe5;
  opacity: 0.7;
  border-radius: 0px;
}

.g-gantt-bar-handle-left {
  left: 0;
  cursor: w-resize;
}

.g-gantt-bar-handle-right {
  right: 0;
  cursor: e-resize;
}

.g-gantt-bar-label img {
  pointer-events: none;
}

.g-gantt-tooltip {
  position: absolute;
  background: black;
  color: white;
  z-index: 3;
  font-size: 0.70em;
  padding: 3px;
  border-radius: 3px;
  transition: opacity 0.2s;
  display: flex;
  align-items: center;
  top: 44px;
}

.g-gantt-tooltip:before {
  content: '';
  position: absolute;
  top: 0;
  left: 10%;
  width: 0;
  height: 0;
  border: 10px solid transparent;
  border-bottom-color: black;
  border-top: 0;
  margin-left: -5px;
  margin-top: -5px;
}

.g-gantt-tooltip>.color-indicator {
  width: 8px;
  height: 8px;
  border-radius: 100%;
  margin-right: 4px;
}

.fade-enter-active {
  animation: fade-in .3s;
}

.fade-leave-active {
  animation: fade-in .3s reverse;
}

@keyframes fade-in {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}
</style>