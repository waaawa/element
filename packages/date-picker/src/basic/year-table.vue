<template>
  <table @click="handleYearTableClick" @mouseover="handleYearTalbeHover" class="el-year-table">
    <tbody>
      <tr v-for="(row,ri) in grid" :key="`row${ri}`">
        <td class="available" v-for="(col, ci) in row" :key="`column${ci}`" 
        :class="[getCellStyle(startYear + col), style[col]]">
          <div>
            <a class="cell">{{ startYear + col }}</a>
          </div>
        </td>
      </tr>
    <!-- <tr>
      <td class="available" :class="getCellStyle(startYear + 0)">
        <a class="cell">{{ startYear }}</a>
      </td>
      <td class="available" :class="getCellStyle(startYear + 1)">
        <a class="cell">{{ startYear + 1 }}</a>
      </td>
      <td class="available" :class="getCellStyle(startYear + 2)">
        <a class="cell">{{ startYear + 2 }}</a>
      </td>
      <td class="available" :class="getCellStyle(startYear + 3)">
        <a class="cell">{{ startYear + 3 }}</a>
      </td>
    </tr>
    <tr>
      <td class="available" :class="getCellStyle(startYear + 4)">
        <a class="cell">{{ startYear + 4 }}</a>
      </td>
      <td class="available" :class="getCellStyle(startYear + 5)">
        <a class="cell">{{ startYear + 5 }}</a>
      </td>
      <td class="available" :class="getCellStyle(startYear + 6)">
        <a class="cell">{{ startYear + 6 }}</a>
      </td>
      <td class="available" :class="getCellStyle(startYear + 7)">
        <a class="cell">{{ startYear + 7 }}</a>
      </td>
    </tr>
    <tr>
      <td class="available" :class="getCellStyle(startYear + 8)">
        <a class="cell">{{ startYear + 8 }}</a>
      </td>
      <td class="available" :class="getCellStyle(startYear + 9)">
        <a class="cell">{{ startYear + 9 }}</a>
      </td>
      <td></td>
      <td></td>
    </tr> -->
    </tbody>
  </table>
</template>

<script type="text/babel">
  import { hasClass } from 'element-ui/src/utils/dom';
  import { isDate, range, nextDate, getDayCountOfYear } from 'element-ui/src/utils/date-util';
  import { arrayFindIndex, coerceTruthyValueToArray, removeRepetition } from 'element-ui/src/utils/util';

  const getStartYear = (v) => {
    return Math.floor(v / 10) * 10;
  };

  const getEndYear = (v) => {
    return Math.ceil(v / 10 + 1) * 10 - 1;
  };

  const datesInYear = year => {
    const numOfDays = getDayCountOfYear(year);
    const firstDay = new Date(year, 0, 1);
    return range(numOfDays).map(n => nextDate(firstDay, n));
  };

  const removeFromArray = function(arr, pred) {
    const idx = typeof pred === 'function'
      ? arrayFindIndex(arr, pred)
      : arr.indexOf(pred);
    return idx >= 0
      ? [...arr.slice(0, idx), ...arr.slice(idx + 1)]
      : arr;
  };

  export default {
    props: {
      disabledDate: {},
      value: {},
      defaultValue: {
        validator(val) {
          // null or valid Date Object
          return val === null || isDate(val) || (Array.isArray(val) && val.every(isDate));
        }
      },
      selectionMode: {
        default: 'year'
      },
      date: {},
      minDate: null,
      maxDate: null,
      visible: {
        type: Boolean,
        require: true
      },
      rangeState: {
        default() {
          return {
            endDate: null,
            selecting: false
          };
        }
      }
    },
    data() {
      return {
        grid: [
          [0, 1, 2, 3],
          [4, 5, 6, 7],
          [8, 9]
        ],
        style: {}
      };
    },
    computed: {
      startYear() {
        return getStartYear(this.date.getFullYear());
      }
    },
    created() {
      this.initStyle();
    },
    watch: {
      visible(n) {
        this.initStyle();
        this.setStyle(this);
        if (!n) {
          this.rangeState.selecting = false;
        }
      },
      startYear(n, o) {
        if (n === o) return;
        this.initStyle();
        this.setStyle(this);
      }
    },
    methods: {
      handleYearTalbeHover(event) {
        if (this.selectionMode !== 'range') return;
        const target = event.target;
        const isCell = hasClass(target, 'cell');
        const year = target.textContent || target.innerText;
        const newVal = new Date(year);
        let param = null;
        if (isCell && this.minDate && !this.maxDate) {
          if (this.minDate > newVal) {
            param = { minDate: newVal, maxDate: this.minDate };
          } else {
            param = { minDate: this.minDate, maxDate: newVal };
          }
          this.initStyle();
          this.setStyle(param);
          this.$forceUpdate();
        }
      },
      initStyle() {
        this.grid.flat(Infinity).forEach(e => {
          this.style[e] = {};
        });
      },
      getCellStyle(year) {
        const style = {};
        const today = new Date();

        if (this.selectionMode === 'range') {
          if (this.minDate && this.maxDate) {
            style.current = false;
          } else {
            style.current = false;
          }
        } else {
          style.current = arrayFindIndex(coerceTruthyValueToArray(this.value), date => date.getFullYear() === year) >= 0;
        }
        style.disabled = typeof this.disabledDate === 'function'
          ? datesInYear(year).every(this.disabledDate)
          : false;
        style.today = today.getFullYear() === year;
        style.default = this.defaultValue && (Array.isArray(this.defaultValue)
          ? this.defaultValue.some(e => e.getFullYear() === year)
          : this.defaultValue.getFullYear() === year);

        return style;
      },
      setStyle({ minDate, maxDate }) {
        let begin = null;
        let end = null;
        if (minDate) {
          begin = String(minDate.getFullYear());
          if (getStartYear(begin) === this.startYear) {
            this.style[begin.charAt(3)]['start-date'] = true;
          }
        }
        if (maxDate) {
          end = String(maxDate.getFullYear());
          if (getStartYear(end) === this.startYear) {
            this.style[end.charAt(3)]['end-date'] = true;
          }
        }
        if (begin && end) {
          let flag = false;
          let s = 0;
          let e = 9;
          const a = this.startYear;
          const b = getEndYear(this.startYear);
          if (begin >= a && begin <= b) {
            s = +begin.charAt(3);
            if (end <= b) e = +end.charAt(3);
          } else if (end >= a && end <= b) {
            e = end.charAt(3);
            if (begin >= a) s = +end.charAt(3);
          } else if (begin < a && end > b) {
            flag = false;
          } else {
            flag = true;
          }
          if (flag) return;
          for (; s <= e; s++) {
            this.style[s]['in-range'] = true;
          }
        }
      },
      handleYearTableClick(event) {
        const target = event.target;
        let selected = false;
        if (target.tagName === 'A') {
          selected = hasClass(target.parentNode.parentNode, 'current');
          if (hasClass(target.parentNode, 'disabled')) return;
          const year = target.textContent || target.innerText;
          if (this.selectionMode === 'years') {
            const value = removeRepetition((this.value || []).map(e => e instanceof Date ? e.getFullYear() : e));
            const newVal = selected
              ? removeFromArray(value, date => date === Number(year))
              : [...value, year];
            this.$emit('pick', newVal);
          } else if (this.selectionMode === 'range') {
            this.initStyle();
            const newVal = new Date(year);
            let param = null;
            if (!this.rangeState.selecting) {
              param = { minDate: newVal, maxDate: null };
              this.$emit('pick', param, false);
              this.rangeState.selecting = true;
            } else {
              param = null;
              if (this.minDate > newVal) {
                param = { minDate: newVal, maxDate: this.minDate };
              } else {
                param = { minDate: this.minDate, maxDate: newVal };
              }
              this.$emit('pick', param);
              this.rangeState.selecting = false;
            }
            this.setStyle(param);
          } else {
            this.$emit('pick', Number(year));
          }
        }
      }
    }
  };
</script>
