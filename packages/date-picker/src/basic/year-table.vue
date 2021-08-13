<template>
  <table @click="handleYearTableClick" class="el-year-table">
    <tbody>
    <tr>
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
    </tr>
    </tbody>
  </table>
</template>

<script type="text/babel">
  import { hasClass } from 'element-ui/src/utils/dom';
  import { isDate, range, nextDate, getDayCountOfYear } from 'element-ui/src/utils/date-util';
  import { arrayFindIndex, coerceTruthyValueToArray, removeRepetition } from 'element-ui/src/utils/util';

  const datesInYear = year => {
    const numOfDays = getDayCountOfYear(year);
    const firstDay = new Date(year, 0, 1);
    return range(numOfDays).map(n => nextDate(firstDay, n));
  };

  const removeFromArray = function(arr, pred) {
    const idx = typeof pred === 'function' ? arrayFindIndex(arr, pred) : arr.indexOf(pred);
    return idx >= 0 ? [...arr.slice(0, idx), ...arr.slice(idx + 1)] : arr;
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
      date: {}
    },

    computed: {
      startYear() {
        return Math.floor(this.date.getFullYear() / 10) * 10;
      }
    },

    methods: {
      getCellStyle(year) {
        const style = {};
        const today = new Date();

        style.disabled = typeof this.disabledDate === 'function'
          ? datesInYear(year).every(this.disabledDate)
          : false;
        style.current = arrayFindIndex(coerceTruthyValueToArray(this.value), date => date.getFullYear() === year) >= 0;
        style.today = today.getFullYear() === year;
        style.default = this.defaultValue && (Array.isArray(this.defaultValue)
          ? this.defaultValue.some(e => e.getFullYear() === year)
          : this.defaultValue.getFullYear() === year);

        return style;
      },

      handleYearTableClick(event) {
        const target = event.target;
        let selected = false;
        if (target.tagName === 'A') {
          selected = target.parentNode.classList.contains('current');
          if (hasClass(target.parentNode, 'disabled')) return;
          const year = target.textContent || target.innerText;
          if (this.selectionMode === 'years') {
            const value = removeRepetition((this.value || []).map(e => e instanceof Date ? e.getFullYear() : e));
            const newVal = selected
              ? removeFromArray(value, date => date === Number(year))
              : [...value, year];;
            this.$emit('pick', newVal);
          } else {
            this.$emit('pick', Number(year));
          }
        }
      }
    }
  };
</script>
