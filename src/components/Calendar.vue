<template>
  <div class="container">
    <div>
      <span>{{currentDate.year}}</span>
      <span>{{currentDate.month}}</span>
      <span>{{currentDate.date}}</span>
    </div>
    <header>
      <div class="prevous" v-on:click="decrease">上个月</div>
      <div class="month">{{thisYear + '年' + thisMonth + '月'}}</div>
      <div class="next" v-on:click="increase">下个月</div>
    </header>
    <table
      style="width:90%;"
      @touchstart="handleTouchStart"
      @touchend="handleTouchEnd">
      <thead>
        <tr>
          <th>日</th>
          <th>一</th>
          <th>二</th>
          <th>三</th>
          <th>四</th>
          <th>五</th>
          <th>六</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, rowIndex) in allDates" :key="rowIndex">
          <td
            v-for="(value, key, colIndex) in row"
            :key="colIndex"
            :style="{
              color: value.type=== 'current' &&
              value.year === today.getFullYear() &&
              value.month === today.getMonth() + 1 &&
              value.date === today.getDate() ? 'red' : 'black'}"
            :class="[
              value.type === 'current' ? 'current' : value.type === 'prev' ? 'prev' : 'post',
              value.year === currentDate.year &&
              value.month === currentDate.month &&
              value.date === currentDate.date && 'active'
              ]"
            @click="handleDateSelect(value)"
          >
            {{value.date}}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<style scoped>
.container {
  font-size: 20px;
}
.prevous {
  font-size: 1.2em;
  display: inline-block;
}
.year {
  font-size: 1.2em;
  display: inline-block;
}
.active {
  text-decoration: underline;
}
.month {
  font-size: 1.2em;
  display: inline-block;
}
.next {
  font-size: 1.2em;
  display: inline-block;
}
.prev {
  background: #8f8f8f;
}
.current {
  background: #fff;
}
.post {
  background: lightgreen;
}
</style>

<script>
export default {
  name: 'Calendar',
  data() {
    const now = new Date();
    const today = now;
    const year = now.getFullYear();
    const month = now.getMonth() + 1;
    const date = now.getDate();
    return {
      today,
      currentDate: {
        year,
        month,
        date,
      },
      thisYear: year,
      thisMonth: month,
      startX: 0,
      startY: 0,
      endX: 0,
      endY: 0,
    };
  },
  computed: {
    allDates() {
      return this.getDatesByMonth(this.thisYear, this.thisMonth);
    },
  },
  methods: {
    increase() {
      if (this.thisMonth === 12) {
        this.thisYear += 1;
        this.thisMonth = 1;
      } else {
        this.thisMonth += 1;
      }
    },
    decrease() {
      if (this.thisMonth === 1) {
        this.thisYear -= 1;
        this.thisMonth = 12;
      } else {
        this.thisMonth -= 1;
      }
    },
    handleTouchStart(e) {
      const currentPoint = e.changedTouches[0];
      this.startX = currentPoint.clientX;
      this.startY = currentPoint.clientY;
    },
    handleTouchEnd(e) {
      const currentPoint = e.changedTouches[0];
      this.endX = currentPoint.clientX;
      this.endY = currentPoint.clientY;
      const disX = this.endX - this.startX;
      const disY = this.endY - this.startY;
      if (disX > 10 && (disY < 10 && disY > -10)) {
        e.stopPropagation();
        this.decrease();
      }
      if (disX < -10 && (disY < 10 && disY > -10)) {
        e.stopPropagation();
        this.increase();
      }
      this.startX = 0;
      this.startY = 0;
      this.endX = 0;
      this.endY = 0;
    },
    handleDateSelect(value) {
      const { year, month, date } = value;
      this.thisYear = year;
      this.thisMonth = month;
      this.currentDate = {
        year,
        month,
        date,
      };
    },
    getDatesByMonth: (year, originMonth) => {
      const month = originMonth - 1;
      const prevousMonth = {
        lastDate: new Date(year, month, 0).getDate(),
      };

      const currentLastDate = new Date(
        month === 11 ? year + 1 : year, month === 11 ? 0 : month + 1, 0,
      ).getDate();
      const currentMonth = {
        lastDate: currentLastDate,
        firstDatePosition: new Date(year, month, 1).getDay(),
        lastDatePosition: new Date(year, month, currentLastDate).getDay(),
      };


      const dateCount = currentMonth.firstDatePosition
                      + currentMonth.lastDate
                      + (7 - currentMonth.lastDatePosition - 1);

      const allDates = [];

      for (let i = 0; i < dateCount / 7; i += 1) {
        allDates[i] = [];
      }

      let rowNum = 0;
      let colNum = 0;

      const pushDate = (currentDate) => {
        allDates[rowNum][colNum] = currentDate;
        colNum += 1;
        if (colNum === 7) {
          colNum = 0;
          rowNum += 1;
        }
      };

      // 0 1 2 3 4 5 6
      // 30-5-0    30-5+5
      // 252627282930

      for (let i = 0; i < currentMonth.firstDatePosition; i += 1) {
        const currentDate = {
          type: 'prev',
          year: originMonth === 1 ? year - 1 : year,
          month: originMonth === 1 ? 12 : originMonth - 1,
          date: prevousMonth.lastDate - (currentMonth.firstDatePosition - 1) + i,
        };

        pushDate(currentDate);
      }

      for (let i = 1; i <= currentMonth.lastDate; i += 1) {
        const currentDate = {
          type: 'current',
          year,
          month: originMonth,
          date: i,
        };
        pushDate(currentDate);
      }

      // 0 1 2 3 4 5 6
      //     2-1     6-1
      // 30310102030405

      for (let i = currentMonth.lastDatePosition + 1; i <= 6; i += 1) {
        const currentDate = {
          type: 'post',
          year: originMonth === 12 ? year + 1 : year,
          month: originMonth === 12 ? 1 : originMonth + 1,
          date: i - currentMonth.lastDatePosition,
        };
        pushDate(currentDate);
      }

      return allDates;
    },
  },
};
</script>
