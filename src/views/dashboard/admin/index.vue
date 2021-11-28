<template>
  <div class="dashboard-editor-container">
    <el-row :gutter="30">
      <el-col :xs="24" :sm="24" :lg="14">
        <div class="chart">
          <panel-group @handleSetLineChartData="handleSetLineChartData" />
          <line-chart :chart-data="lineChartData" />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="10">
        <div class="chart">
          <aum-chart />
        </div>
      </el-col>
    </el-row>
    <el-row :gutter="32">
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <age />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <gender />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <level />
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>

import PanelGroup from './components/PanelGroup'
import LineChart from './components/LineChart'
import AumChart from './components/AumChart'
import Age from './components/Age'
import Gender from './components/Gender'
import Level from './components/Level'

const lineChartData = {
  newVisitis: {
    expectedData: [100, 120, 161, 134, 105, 160],
    actualData: [120, 82, 91, 154, 162, 140]
  },
  messages: {
    expectedData: [200, 192, 120, 144, 160, 130],
    actualData: [180, 160, 151, 106, 145, 150]
  },
  purchases: {
    expectedData: [80, 100, 121, 104, 105, 90],
    actualData: [120, 90, 100, 138, 142, 130]
  },
  shoppings: {
    expectedData: [130, 140, 141, 142, 145, 150],
    actualData: [120, 82, 91, 154, 162, 140]
  }
}

export default {
  name: 'DashboardAdmin',
  components: {
    PanelGroup,
    LineChart,
    AumChart,
    Age,
    Gender,
    Level
  },
  data() {
    return {
      lineChartData: lineChartData.newVisitis
    }
  },
  methods: {
    handleSetLineChartData(type) {
      this.lineChartData = lineChartData[type]
    }
  }
}
</script>

<style lang="scss" scoped>
  .dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .github-corner {
    position: absolute;
    top: 0px;
    border: 0;
    right: 0;
  }

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }

  .chart{
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width:1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>
