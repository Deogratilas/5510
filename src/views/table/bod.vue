<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.title" placeholder="Bond_No" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <!-- <el-select v-model="listQuery.importance" placeholder="Imp" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in importanceOptions" :key="item" :label="item" :value="item" />
      </el-select> -->
      <!-- <el-select v-model="listQuery.type" placeholder="Type" clearable class="filter-item" style="width: 130px">
        <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key" />
      </el-select> -->
      <!-- <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item" @change="handleFilter">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" />
      </el-select> -->
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
        Search
      </el-button>
      <!-- <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        Add
      </el-button> -->
      <el-button v-waves :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload">
        Export
      </el-button>
      <!-- <el-checkbox v-model="showReviewer" class="filter-item" style="margin-left:15px;" @change="tableKey=tableKey+1">
        reviewer
      </el-checkbox> -->
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
    >
      <el-table-column label="Bond_No" prop="id" sortable="custom" align="center" width="150px" :class-name="getSortClass('id')">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="meanrate" width="200px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.timestamp }}</span>
        </template>
      </el-table-column>
      <el-table-column label="standard_variance" min-width="150px">
        <template slot-scope="{row}">
          <span> {{ row.title }}</span>
          <!-- <el-tag>{{ row.type | typeFilter }}</el-tag> -->
        </template>
      </el-table-column>
      <el-table-column label="sharp" width="110px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.author }}</span>
        </template>
      </el-table-column>
      <!-- <el-table-column v-if="showReviewer" label="Reviewer" width="110px" align="center">
        <template slot-scope="{row}">
          <span style="color:red;">{{ row.reviewer }}</span>
        </template>
      </el-table-column> -->
      <el-table-column label="β" width="80px">
        <template slot-scope="{row}">
          <span>{{ row.importance }}</span>
        </template>
      </el-table-column>
      <el-table-column label="type" align="center" width="100">
        <template slot-scope="{row}">
          <span v-if="row.pageviews" class="link-type" @click="handleFetchPv(row.pageviews)">{{ row.pageviews }}</span>
          <span v-else>0</span>
        </template>
      </el-table-column>
      <!-- <el-table-column label="New buyer" align="center" width="100">
        <template slot-scope="{row}">
          <span v-if="row.pageviews" class="link-type" @click="handleFetchPv(row.pageviews)">{{ row.pageviews-177 }}</span>
          <span v-else>0</span>
        </template>
      </el-table-column> -->
      <!-- <el-table-column label="Status" class-name="status-col" width="100">
        <template slot-scope="{row}">
          <el-tag :type="row.status | statusFilter">
            {{ row.status }}
          </el-tag>
        </template>
      </el-table-column> -->
      <!-- <el-table-column label="Actions" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            Edit
          </el-button>
          <el-button v-if="row.status!='published'" size="mini" type="success" @click="handleModifyStatus(row,'published')">
            Publish
          </el-button>
          <el-button v-if="row.status!='draft'" size="mini" @click="handleModifyStatus(row,'draft')">
            Draft
          </el-button>
          <el-button v-if="row.status!='deleted'" size="mini" type="danger" @click="handleDelete(row,$index)">
            Delete
          </el-button>
        </template>
      </el-table-column> -->
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="Type" prop="type">
          <el-select v-model="temp.type" class="filter-item" placeholder="Please select">
            <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name" :value="item.key" />
          </el-select>
        </el-form-item>
        <el-form-item label="Date" prop="timestamp">
          <el-date-picker v-model="temp.timestamp" type="datetime" placeholder="Please pick a date" />
        </el-form-item>
        <el-form-item label="Title" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="Status">
          <el-select v-model="temp.status" class="filter-item" placeholder="Please select">
            <el-option v-for="item in statusOptions" :key="item" :label="item" :value="item" />
          </el-select>
        </el-form-item>
        <el-form-item label="Imp">
          <el-rate v-model="temp.importance" :colors="['#99A9BF', '#F7BA2A', '#FF9900']" :max="3" style="margin-top:8px;" />
        </el-form-item>
        <el-form-item label="Remark">
          <el-input v-model="temp.remark" :autosize="{ minRows: 2, maxRows: 4}" type="textarea" placeholder="Please input" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          Confirm
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'ComplexTable',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    }
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: '+id'
      },
      importanceOptions: [1, 2, 3, 4, 5],
      calendarTypeOptions,
      sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
      statusOptions: ['published', 'draft', 'deleted'],
      showReviewer: false,
      temp: {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        type: '',
        status: 'published'
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
      downloadLoading: false
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      fetchList(this.listQuery).then(response => {
        // this.list = response.data.items
        this.list = [{ id: 1, timestamp: 0.0246, author: 0.0013, title: '7.02', importance: 0, pageviews: 1 },
          { id: 2, timestamp: 0.0236, author: 0.0012, title: '6.09', importance: 0, pageviews: 1 },
          { id: 3, timestamp: 0.0248, author: 0.0012, title: '7.43', importance: 0, pageviews: 1 },
          { id: 4, timestamp: 0.0249, author: 0.0012, title: '8.42', importance: 0, pageviews: 1 },
          { id: 5, timestamp: 0.0245, author: 0.0012, title: '8.07', importance: 0, pageviews: 1 },
          { id: 6, timestamp: 0.375, author: 0.2353, title: '1.4', importance: 12.88, pageviews: 2 },
          { id: 7, timestamp: 0.0597, author: 0.2121, title: '0.34', importance: -4.14, pageviews: 2 },
          { id: 8, timestamp: 0.0858, author: 0.2238, title: '0.37', importance: -41.28, pageviews: 2 },
          { id: 9, timestamp: 0.3161, author: 0.2543, title: '1.04', importance: 8.36, pageviews: 3 },
          { id: 10, timestamp: 0.32, author: 0.2422, title: '1.14', importance: 11.56, pageviews: 3 },
          { id: 11, timestamp: 0.1745, author: 0.1213, title: '1.71', importance: 4.83, pageviews: 3 },
          { id: 12, timestamp: 0.1672, author: 0.2138, title: '0.72', importance: 9.3, pageviews: 3 },
          { id: 13, timestamp: 0.0997, author: 0.1977, title: '0.46', importance: 4.85, pageviews: 3 },
          { id: 14, timestamp: 0.3142, author: 0.27, title: '1.03', importance: 17.27, pageviews: 3 },
          { id: 15, timestamp: 0.2141, author: 0.2302, title: '0.98', importance: 7.16, pageviews: 3 },
          { id: 16, timestamp: 0.2986, author: 0.2038, title: '1.26', importance: 11.77, pageviews: 3 },
          { id: 17, timestamp: 0.3908, author: 0.2123, title: '1.57', importance: 8.89, pageviews: 3 },
          { id: 18, timestamp: 0.1266, author: 0.2718, title: '0.5', importance: 4.64, pageviews: 3 },
          { id: 19, timestamp: 0.1696, author: 0.1701, title: '0.88', importance: 15.1, pageviews: 3 },
          { id: 20, timestamp: 0.587, author: 0.2677, title: '1.75', importance: 4.75, pageviews: 2 },
          { id: 21, timestamp: 0.6096, author: 0.2867, title: '1.71', importance: 4.93, pageviews: 2 },
          { id: 22, timestamp: 0.2342, author: 0.2128, title: '0.95', importance: 16.79, pageviews: 2 },
          { id: 23, timestamp: 0.4858, author: 0.2484, title: '1.59', importance: 5.58, pageviews: 2 },
          { id: 24, timestamp: 0.2765, author: 0.2068, title: '1.19', importance: 23.64, pageviews: 2 },
          { id: 25, timestamp: 0.4145, author: 0.2425, title: '1.42', importance: 6.87, pageviews: 2 },
          { id: 26, timestamp: 0.3026, author: 0.2096, title: '1.24', importance: 11.45, pageviews: 2 },
          { id: 27, timestamp: 0.6173, author: 0.3267, title: '1.56', importance: 5.81, pageviews: 2 },
          { id: 28, timestamp: 0.5326, author: 0.2806, title: '1.57', importance: 6.11, pageviews: 2 },
          { id: 29, timestamp: 0.6751, author: 0.3165, title: '1.71', importance: 4.75, pageviews: 2 },
          { id: 30, timestamp: 0.4027, author: 0.2297, title: '1.46', importance: 7.08, pageviews: 2 },
          { id: 31, timestamp: 0.5549, author: 0.2618, title: '1.67', importance: 4.48, pageviews: 2 },
          { id: 32, timestamp: 0.6337, author: 0.2512, title: '1.98', importance: 4.28, pageviews: 2 },
          { id: 33, timestamp: 0.4078, author: 0.2549, title: '1.36', importance: 8.43, pageviews: 2 },
          { id: 34, timestamp: 0.4581, author: 0.2306, title: '1.65', importance: 6.6, pageviews: 2 },
          { id: 35, timestamp: 0.4888, author: 0.2571, title: '1.54', importance: 5.43, pageviews: 2 },
          { id: 36, timestamp: 0.439, author: 0.2589, title: '1.46', importance: 9.22, pageviews: 2 },
          { id: 37, timestamp: 0.389, author: 0.1851, title: '1.72', importance: 6.29, pageviews: 2 },
          { id: 38, timestamp: 0.4659, author: 0.2575, title: '1.51', importance: 6.81, pageviews: 2 },
          { id: 39, timestamp: 0.6691, author: 0.2922, title: '1.8', importance: 4.27, pageviews: 2 },
          { id: 40, timestamp: 0.7381, author: 0.2789, title: '2.21', importance: 6.06, pageviews: 2 },
          { id: 41, timestamp: 0.0567, author: 0.0313, title: '1.17', importance: 5.29, pageviews: 4 },
          { id: 42, timestamp: 0.1024, author: 0.0512, title: '1.6', importance: 12.56, pageviews: 4 },
          { id: 43, timestamp: 0.0443, author: 0.014, title: '1.3', importance: 20.22, pageviews: 4 },
          { id: 44, timestamp: 0.1213, author: 0.0579, title: '1.68', importance: 32.73, pageviews: 4 },
          { id: 45, timestamp: 0.0858, author: 0.0481, title: '1.34', importance: 11.4, pageviews: 4 },
          { id: 46, timestamp: 0.1292, author: 0.0816, title: '1.31', importance: 22.76, pageviews: 4 },
          { id: 47, timestamp: 0.1052, author: 0.0823, title: '1.03', importance: 12.9, pageviews: 4 },
          { id: 48, timestamp: 0.1187, author: 0.0849, title: '1.24', importance: 7.75, pageviews: 4 },
          { id: 49, timestamp: 0.0472, author: 0.0166, title: '1.89', importance: 2.81, pageviews: 4 },
          { id: 50, timestamp: 0.1024, author: 0.064, title: '1.27', importance: 13.82, pageviews: 4 }
        ]
        this.total = response.data.total

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false
        }, 1.5 * 1000)
      })
    },
    handleFilter() {
      // this.listQuery.page = 1
      // this.getList()
      this.list = [{ id: 1, timestamp: 0.0246, author: 0.0013, title: '7.02', importance: 0, pageviews: 1 }]
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: '操作Success',
        type: 'success'
      })
      row.status = status
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        status: 'published',
        type: ''
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          this.temp.author = 'vue-element-admin'
          createArticle(this.temp).then(() => {
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateArticle(tempData).then(() => {
            const index = this.list.findIndex(v => v.id === this.temp.id)
            this.list.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleDelete(row, index) {
      this.$notify({
        title: 'Success',
        message: 'Delete Successfully',
        type: 'success',
        duration: 2000
      })
      this.list.splice(index, 1)
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
        const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
        const data = this.formatJson(filterVal)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal) {
      return this.list.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}` ? 'ascending' : 'descending'
    }
  }
}
</script>
