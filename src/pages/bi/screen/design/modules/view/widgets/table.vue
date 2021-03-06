<template>
  <div
    :class="[this.config.table.horizontal ? 'chart-table-horizontal' : 'chart-table-vertical']"
    ref="chart"
    class="cross-chart-table"
  >
    <q-markup-table flat :style="config.table.tableStyle">
      <thead>
        <tr>
          <th :style="thStyle" v-for="(v,idx) in list.head" :key="idx">{{v}}</th>
        </tr>
      </thead>
      <tbody :style="tableStyle()">
        <tr v-for="(v,i) in list.body" :key="i">
          <td :style="tdStyle" v-for="(d,idx) in v" :key="idx">{{d}}</td>
        </tr>
      </tbody>
    </q-markup-table>
  </div>
</template>

<script>
import { debounce } from 'quasar';

export default {
  name: 'ChartTable',
  props: {
    config: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      data: [],
      interval: this.config.interval,
      ti: 0,
      loading: false,
      tableData: [],
    };
  },
  methods: {
    tableStyle() {
      const resultStyle = { };
      if (this.config.table.loop) {
        resultStyle.animationDuration = `${this.config.table.scrolldelay}s`;
        resultStyle.animationTimingFunction = 'linear';
        resultStyle.animationIterationCount = 'infinite';
        resultStyle.animationDirection = 'alternate';
        resultStyle.animationDelay = '2s';
        if (this.config.table.alternate) {
          resultStyle.animationDirection = 'alternate';
        } else {
          resultStyle.animationDirection = 'normal';
        }
        if (this.config.table.direction === 'left') {
          resultStyle.animationName = 'tableLoopLeft';
        } else if (this.config.table.direction === 'right') {
          resultStyle.animationName = 'tableLoopRight';
        } else if (this.config.table.direction === 'up') {
          resultStyle.animationName = 'tableLoopUp';
        } else if (this.config.table.direction === 'down') {
          resultStyle.animationName = 'tableLoopDown';
        }
      }
      return resultStyle;
    },
    loop() {
      clearTimeout(this.ti);
      this.interval -= 1;
      if (this.interval < 1) {
        this.getData();
      } else {
        this.ti = setTimeout(() => {
          this.loop();
        }, 999);
      }
    },
    confirmLoop() {
      if (this.loopHandler) {
        this.interval = this.config.interval - (-1);
        this.loop();
      } else {
        clearTimeout(this.ti);
      }
    },
    getOrders() {
      const orders = [];
      this.config.orders.forEach((o) => {
        orders.push({
          column: o.name,
          direction: o.sort,
        });
      });
      return orders;
    },
    doLoadData() {
      this.loading = true;
      if (this.config.viewId && (this.config.cols.length > 0 || this.config.rows.length > 0)) {
        this.$axios.post(`/bi/view/getTableData/${this.config.viewId}`, {
          aggregators: this.config.rows.map((v) => ({
            column: v.name.split('@')[0],
            alias: v.field.alias,
            func: v.agg,
          })),
          allParam: this.allParam,
          cache: false,
          expired: 0,
          filters: this.config.filters,
          groups: this.config.cols,
          nativeQuery: false,
          horizontal: this.config.table.horizontal,
          orders: this.getOrders(),
          pageNo: 1,
          pageSize: this.config.length,
        }).then(({ result }) => {
          this.data = result;
        }).finally(() => {
          this.loading = false;
        });
      }
    },
  },
  created() {
    this.getData = debounce(this.doLoadData, 500);
  },
  mounted() {
    if (this.config.viewId) {
      this.getData();
    }
  },
  watch: {
    config: {
      deep: true,
      handler() {
        if (this.config.needResize) {
          this.config.needResize = false;
        }
        this.getData();
      },
    },
    'config.cols': {
      handler() {
        this.getData();
      },
      deep: true,
    },
    'config.rows': {
      handler() {
        this.getData();
      },
      deep: true,
    },
    'config.loop': {
      handler(n, o) {
        if (n !== o) {
          this.confirmLoop();
        }
      },
    },
    'config.interval': {
      handler(n, o) {
        if (n !== o) {
          this.confirmLoop();
        }
      },
    },
    'config.length': {
      handler(n, o) {
        if (n !== o) {
          this.getData();
        }
      },
    },
  },
  computed: {
    list() {
      return this.data;
    },
    thStyle() {
      const tableHead = {
        ...this.config.table.tableHead,
        fontSize: `${this.config.table.tableHead.fontSize}px`,
        borderWidth: `${this.config.table.tableHead.borderWidth}px`,
        opacity: this.config.table.tableHead.opacity / 100,
      };
      if (tableHead.backgroundColor === '') {
        tableHead.backgroundColor = 'transparent';
      }
      return tableHead;
    },
    tdStyle() {
      const tableBody = {
        ...this.config.table.tableBody,
        fontSize: `${this.config.table.tableBody.fontSize}px`,
        borderWidth: `${this.config.table.tableBody.borderWidth}px`,
        opacity: this.config.table.tableBody.opacity / 100,
      };
      if (tableBody.backgroundColor === '') {
        tableBody.backgroundColor = 'transparent';
      }
      return tableBody;
    },
  },
  beforeDestroy() {
    clearTimeout(this.ti);
  },
};
</script>

<style lang="stylus">
.bg-transparent-table
  background-color transparent
.cross-chart-table
  .q-table
    thead
      th,td
        position sticky
        opacity 1
        z-index: 1
      tr
        th
          position sticky
          top 0
          z-index 1
@keyframes tableLoopLeft {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-200px);
  }
}
@keyframes tableLoopRight {
  0% {
    transform: translateX(-200px);
  }
  100% {
    transform: translateX(0);
  }
}
@keyframes tableLoopUp {
  0% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(-100px);
  }
}
@keyframes tableLoopDown {
  0% {
    transform: translateY(-100px);
  }
  100% {
    transform: translateY(0);
  }
}

.chart-table-horizontal {
  th:first-child {
    position: sticky;
    left: 0;
  }
}

.chart-table-vertical {
  th:first-child {
    position: sticky;
    left: 0;
  }
}
</style>
