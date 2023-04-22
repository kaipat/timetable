<template>
  <div class="app">
    <div class="time-picker">
      <div class="time-picker-header-row">
        <div class="row-title">星期/时间</div>
        <div>
          <div class="time-range">
            <div class="rang-span">00:00 - 12:00</div>
            <div class="rang-span">12:00 - 24:00</div>
          </div>
          <div class="time-unit-row">
            <div
              v-for="hour in HOURS"
              :key="hour"
              class="time-unit">
              {{ hour }}
            </div>
          </div>
        </div>
      </div>
      <div
        v-for="(row, rowIndex) in rows"
        :key="rowIndex"
        class="time-picker-row"
        @mousedown="handleStartCheck"
        @mouseup="handleEndCheck"
        @mousemove="handleCheck">
        <div class="row-title">{{ row.title }}</div>
        <div
          v-for="(col, colIndex) in row.units"
          :key="`${colIndex},${rowIndex}`"
          :data-x="colIndex"
          :data-y="rowIndex"
          :class="['row-unit', col.checked ? 'active' : '']">
        </div>
      </div>
    </div>
  </div>
</template>

<script>

const WEEKS = ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期日"];

const HOURS = Array.from({length: 24}, (_, i) => i);

const TIMES = Array.from({length: 48}, (_, i) => i);

const ROWS = WEEKS.map(
 (week, weekIndex) => ({
    title: week,
    units: TIMES.map((_, timeIndex) => ({ checked: false, position: {x: timeIndex,y: weekIndex} }))
  })
);

const ACTION = {
  checking: false,
  target: null,
  start: null,
  end: null,
  type: "check" // [check, uncheck]
}

export default {
  name: 'App',
  data() {
    return {
      WEEKS, HOURS,
      rows: JSON.parse(JSON.stringify(ROWS)),
      action: { ...ACTION },
    }
  },
  methods: {
    direction(start, end, target) {
      if (start.x > end.x && start.y > end.y) {
        // console.log("left top");
        return target.x >= end.x && target.x <= start.x && target.y >= end.y && target.y <= start.y
      } else if (start.x < end.x && start.y > end.y) {
        // console.log("right top");
        return target.x <= end.x && target.x >= start.x && target.y >= end.y && target.y <= start.y
      } else if (start.x > end.x && start.y < end.y) {
        // console.log("left bottom");
        return target.x >= end.x && target.x <= start.x && target.y <= end.y && target.y >= start.y
      } else if (start.x < end.x && start.y < end.y) {
        // console.log("right bottom");
        return target.x <= end.x && target.x >= start.x && target.y <= end.y && target.y >= start.y
      } else if (start.x === end.x && start.y < end.y) {
        // console.log("bottom");
        return target.x === end.x && target.x === start.x && target.y <= end.y && target.y >= start.y
      } else if (start.x === end.x && start.y > end.y) {
        // console.log("top");
        return target.x === end.x && target.x === start.x && target.y >= end.y && target.y <= start.y
      } else if (start.x < end.x && start.y === end.y) {
        // console.log("right");
        return target.x <= end.x && target.x >= start.x && target.y === end.y && target.y === start.y
      } else if (start.x > end.x && start.y === end.y) {
        // console.log("left");
        return target.x >= end.x && target.x <= start.x && target.y === end.y && target.y === start.y
      }
    },
    handleCheck(event) {
      if (!this.checking) return;
      if (this.action.target === event.target) return;
      const x = parseInt(event.target.dataset.x, 10);
      const y = parseInt(event.target.dataset.y, 10);
      if (x >= 0 && y >= 0) {
        this.action.target = event.target;
        const { start } = this.action;
        const end = { x, y };
        this.rows.forEach(row => {
          row.units.forEach(unit => {
            if (this.direction(start, end, unit.position)) {
              unit.checked = this.action.type === "check";
            }
          });
        });
      }
    },
    handleStartCheck(event) {
      const x = parseInt(event.target.dataset.x, 10);
      const y = parseInt(event.target.dataset.y, 10);
      if (x >= 0 && y >= 0) {
        this.action.target = event.target;
        this.action.start = { x, y };
        this.rows[y].units[x].checked = !this.rows[y].units[x].checked;
        this.action.type = this.rows[y].units[x].checked ? "check" : "uncheck"
        this.checking = true;
      }
    },
    handleEndCheck() {
      this.checking = false;
    },
  }
}
</script>

<style scoped lang="scss">
$unit-width: 16px;
$unit-height: 3 * $unit-width;
$active-color: #3b82f6;
$border-color: #cbd5e1;
$unit-border-color: #f8fafc;
$header-bg-1: #e2e8f0;
$header-bg-2: #f1f5f9;
$unit-bg-1: #e2e8f0;
$unit-bg-2: #f1f5f9;
.time-picker {
  width: fit-content;
  border: 1px solid $border-color;
}
.time-picker-header-row {
  display: flex;
  border-bottom: 1px solid $border-color;
  user-select: none;
}
.time-range {
  position: relative;
  display: flex;
  border-bottom: 1px solid $border-color;
  &:after {
    position: absolute;
    left: 50%;
    top: 0;
    transform: translate(-50%, 0);
    display: block;
    content: "";
    width: 2px;
    height: $unit-height;
    background: $header-bg-2;
  }
}
.rang-span {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24 * $unit-width;
  height: $unit-height;
  background: $header-bg-1;
}
.time-unit-row {
  display: flex;
}
.time-unit {
  width: 2 * $unit-width;
  display: flex;
  align-items: center;
  justify-content: center;
  background: $header-bg-2;
}
.time-picker-row {
  display: flex;
  &:not(:last-of-type) {
    border-bottom: 1px solid $border-color;
  }
}
.row-title {
  width: 6 * $unit-width;
  display: flex;
  align-items: center;
  justify-content: center;
  border-right: 1px solid $border-color;
  user-select: none;
}
.row-unit {
  width: $unit-width;
  height: $unit-height;
  background: $unit-bg-1;
  box-sizing: border-box;
  border: 1px solid $unit-border-color;
  cursor: pointer;
  user-select: none;
  &:nth-of-type(2n + 1) {
    background: $unit-bg-2;
  }
  &.active {
    background: $active-color;
  }
}
</style>
