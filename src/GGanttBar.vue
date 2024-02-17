<template>
  <div>
    <div class="g-gantt-bar" ref="g-gantt-bar" style="cursor: pointer; background: rgba(0, 171, 85, 0.7);"
      :style="barStyle">
      <div class="g-gantt-bar-label" @mousedown.left.stop="startDrag($event, bar, 'drag')" @dblclick="showTaskInfo(bar)">
        <span style=""> {{ bar.label }}
        </span>
        <slot name="bar-label" :bar="bar">
          {{ barConfig.label || "" }}
        </slot>
      </div>
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
  name: "GGanttBar",
  props: {
    widthAxis: { type: Number },
    chartStart: { type: String },
    chartEnd: { type: String },
    rectC: { type: [Object, DOMRect] },
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

      draggedTask: null,
      dragEdge: null,
      initialX1: 0,
      initialX2: 0,
      dragX1: 0,
      dragX2: 0
    }
  },
  filters: {
    TimeFilter(value) {
      return moment(value, 'YYYY-MM-DD HH:mm').format("DD-MM-YYYY HH:mm")
    }
  },
  computed: {

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
        background: this.bar?.task_state == 'ESTIMATED' ? '#0177c0!important' : (this.bar?.task_state == 'DONE' ? '#f18a2d' : (this.bar?.task_state == 'CLOSED' ? '#7e349d' : (this.bar?.task_state == 'NEW' ? '#00AB55' : (this.bar?.task_state == 'CANCELED' ? '#e74c3c' : (this.bar?.task_state == 'IN_PROCESS' ? '#FFC107' : '#08aba0'))))),
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
    showTaskInfo(bar) {
      if (bar && bar.id) {
        let url = 'forms/task_view/' + bar.id + '?_target=modal_1100'
        this.$root.navigateTo(url)
      }
    },
    startDrag(event, task, edge) {
      this.isDragging = true;
      this.draggedTask = task;
      this.dragEdge = edge;

      this.initialX1 = this.mapTimeToPosition(this.bar.myStart)
      this.initialX2 = this.mapTimeToPosition(this.bar.myEnd)
      this.dragX1 = event.clientX - this.rectC.left

      document.addEventListener('mousemove', this.handleDrag);
      document.addEventListener('mouseup', this.stopDrag);
    },
    handleDrag(event) {
      if (this.isDragging) {
        this.dragX2 = (event.clientX - this.rectC.left) - this.dragX1
        if (this.dragEdge === 'start') {
          console.log('start')
          this.draggedTask.myStart = this.mapPositionToTime(event.clientX - this.rectC.left)
        } else if (this.dragEdge === 'end') {
          console.log('end')
          this.draggedTask.myEnd = this.mapPositionToTime(event.clientX - this.rectC.left)
        } else if (this.dragEdge === 'drag') {
          console.log('drag')
          this.draggedTask.myStart = this.mapPositionToTime(this.initialX1 + this.dragX2)
          this.draggedTask.myEnd = this.mapPositionToTime(this.initialX2 + this.dragX2)
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
    // ------- Mapping position to time and vice versa ------- 
    mapTimeToPosition(time) {
      let hourDiffFromStart = moment(time, 'YYYY-MM-DD HH:mm').diff(this.chartStartMoment, "hour", true)
      return (hourDiffFromStart / this.getHourCount()) * this.widthAxis
    },
    mapPositionToTime(xPos) {
      let hourDiffFromStart = (xPos / this.widthAxis) * this.getHourCount()
      return this.chartStartMoment.clone().add(hourDiffFromStart, "hours")
    },
  },

}
</script>

<style scoped>
.g-gantt-bar {
  font-family: sans-serif;
  position: absolute;
  top: 8px;
  left: 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
  width: 300px;
  font-weight: 400;
  font-size: 14px;
  height: 28px;
  border-radius: 4px;
  background: #53a653;
  /*   background: #79869c; */
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
</style>