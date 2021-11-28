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
          text: 'Potential Loss Customer Average Last Trading Time',
          left: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{b} : {c}'
        },
        xAxis: {
          data: ['Type 1', 'Type 2', 'Type 3', 'Type 4']
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            data: [509.4, 577.2, 2551.8, 529.3],
            type: 'bar',
            color: '#ffc6dc',
            showBackground: true,
            backgroundStyle: {
              color: 'rgba(180, 180, 180, 0.2)'
            }
          }
        ]
      })
    }
  }
}
</script>
