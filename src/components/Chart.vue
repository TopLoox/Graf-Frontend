<template>
  <div>
    <q-card>
      <q-card-section class="text-h6">
        <div class="graf_text">
          График платежей
        </div>
      </q-card-section>
      <q-card-section>
        <div ref="chart" id="Chart" style="height: 250px;"></div>
      </q-card-section>
      <q-resize-observer @resize="onResize"/>
    </q-card>
  </div>
</template>

<script>

export default {
  name: "Chart",
  props: {
    series: {
      type: Array,
      default() {
        return [];
      }
    },
    xAxis: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  data() {
    return {
      model: false,
      options: {
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#6a7985'
            }
          }
        },
        series: this.series,
        xAxis: this.xAxis,
          dataZoom: [
          {
            type: 'inside',
          },
        ],
        legend: {
          data: ['Email marketing', 'Affiliate advertising', 'Video advertising', 'Direct access', 'Search engine'],
          bottom: 10,
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '15%',
          top:'5%',
          containLabel: true
        },
        yAxis: [
          {
            type: 'value'
          }
        ],
      },
      area_chart: null
    }
  },
  watch: {
    '$q.dark.isActive': function () {
      this.init();
    }
  },
  methods: {
    init() {
      console.log('Connect!')
      let Chart = document.getElementById('Chart');
      echarts.dispose(Chart);
      let theme = this.model ? 'dark' : 'light';
      this.chart = echarts.init(Chart, theme);
      this.chart.setOption(this.options)
    },

    onResize() {
      if (this.chart) {
        this.chart.resize();
      }
    },
  }
}
</script>

<style scoped>

</style>
