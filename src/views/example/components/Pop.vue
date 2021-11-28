<template>
  <div :class="className" :style="{height:height,width:width}" />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons')
import resize from './mixins/resize'

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    }
  },
  data() {
    return {
      chart: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')
      this.chart.setOption({
        title: {
          text: 'Potential Loss Customer Number Changing'
        },
        tooltip: {
          trigger: 'axis'
        },
        legend: {
          data: ['Type 1', 'Type 2', 'Type 3', 'Type 4'],
          left: 'right'
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: ['Aug', 'Sep', 'Oct', 'Nov', 'Dec']
        },
        yAxis: {
          type: 'value',
          axisLabel: {
            show: true,
            interval: 'auto',
            formatter: '{value} %'
          },
          show: true
        },
        series: [
          {
            name: 'Type 1',
            type: 'line',
            stack: 'Total',
            data: [10, -23.6, 32.7, -32.8, 13.5]
          },
          {
            name: 'Type 2',
            type: 'line',
            stack: 'Total',
            data: [-17, 1.4, -12.5, 5.7, 6.1]
          },
          {
            name: 'Type 3',
            type: 'line',
            stack: 'Total',
            data: [12, 14.1, 20.1, -15.4, -19]
          },
          {
            name: 'Type 4',
            type: 'line',
            stack: 'Total',
            data: [3.2, 5.3, -3.6, 1.9, 5.7]
          }
        ]
      })
    }
  }
}
</script>
