<template>
  <div class="createPost-container">
    <el-row :gutter="32">
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <age />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <cust />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <Time />
        </div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :xs="24" :sm="24" :lg="24">
        <div class="big_chart">
          <Assert />
        </div>
      </el-col>
    </el-row>
    <el-row :gutter="32">
      <el-col :xs="24" :sm="24" :lg="12">
        <div class="pop">
          <pop />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="12">
        <div class="pop">
          <Money />
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Age from './Age'
import Cust from './Cust'
import Time from './Time'
import Assert from './Assert'
import Pop from './Pop'
import Money from './Money'
export default {
  components: {
    Money,
    Age,
    Cust,
    Time,
    Assert,
    Pop
  }
}
</script>
<style lang="scss" scoped>

.createPost-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;
}
.chart-wrapper {
  background: #fff;
  padding: 16px 16px 0;
  margin-bottom: 32px;
}
.big_chart {
  background: #fff;
  padding: 16px 16px 0;
  margin-bottom: 32px;
}
.pop {
  background: #fff;
  padding: 16px 16px 0;
  margin-bottom: 32px;
}

@media (max-width:1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>
