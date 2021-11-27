<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.title" placeholder="Custmoer_Name" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <!-- <el-select v-model="listQuery.importance" placeholder="Imp" clearable style="width: 90px" class="filter-item">
        <el-option v-for="item in importanceOptions" :key="item" :label="item" :value="item" />
      </el-select>
      <el-select v-model="listQuery.type" placeholder="Type" clearable class="filter-item" style="width: 130px">
        <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key" />
      </el-select>
      <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item" @change="handleFilter">
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
      <el-table-column label="Customer_No" prop="id" sortable="custom" align="center" width="200px" :class-name="getSortClass('id')">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Last_Operation_Date" width="250px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.timestamp | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Name" min-width="100px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.title }}</span>
          <!-- <el-tag>{{ row.type | typeFilter }}</el-tag> -->
        </template>
      </el-table-column>
      <el-table-column label="Age" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.author }}</span>
        </template>
      </el-table-column>
      <!-- <el-table-column v-if="showReviewer" label="Reviewer" width="110px" align="center">
        <template slot-scope="{row}">
          <span style="color:red;">{{ row.reviewer }}</span>
        </template>
      </el-table-column> -->
      <el-table-column label="Level" width="150px">
        <template slot-scope="{row}">
          <svg-icon v-for="n in + row.importance" :key="n" icon-class="star" class="meta-item__icon" />
        </template>
      </el-table-column>
      <el-table-column label="Deposits" align="center" width="130px">
        <template slot-scope="{row}">
          <span v-if="row.pageviews" class="link-type" @click="handleFetchPv(row.pageviews)">{{ row.pageviews }}</span>
          <span v-else>0</span>
        </template>
      </el-table-column>
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
        console.log(response.data.items)
        this.list = response.data.items
        this.list = [{ id: 3000261844, timestamp: 1599705049558, author: 60, title: 'James', importance: 3, pageviews: 0 },
          { id: 3000261852, timestamp: 1599705049516, author: 20, title: 'Robert', importance: 3, pageviews: 3119 },
          { id: 3000261859, timestamp: 1599705049524, author: 65, title: 'John', importance: 2, pageviews: 17078 },
          { id: 3000261888, timestamp: 1599705049420, author: 29, title: 'Michael', importance: 3, pageviews: 12546 },
          { id: 3000261898, timestamp: 1599705049434, author: 54, title: 'William', importance: 1, pageviews: 42604 },
          { id: 3000261907, timestamp: 1599705049358, author: 49, title: 'David', importance: 2, pageviews: 53150 },
          { id: 3000261917, timestamp: 1599705049408, author: 30, title: 'Richard', importance: 3, pageviews: 44688 },
          { id: 3000261937, timestamp: 1599705049208, author: 40, title: 'Joseph', importance: 5, pageviews: 14196 },
          { id: 3000261941, timestamp: 1599705049390, author: 58, title: 'Thomas', importance: 1, pageviews: 51080 },
          { id: 3000262130, timestamp: 1599705049108, author: 19, title: 'Charles', importance: 3, pageviews: 47205 },
          { id: 3000262158, timestamp: 1599705049258, author: 29, title: 'Christopher', importance: 1, pageviews: 63640 },
          { id: 3000262186, timestamp: 1599705049129, author: 63, title: 'Daniel', importance: 2, pageviews: 105160 },
          { id: 3000262192, timestamp: 1599705049282, author: 47, title: 'Matthew', importance: 2, pageviews: 102300 },
          { id: 3000262206, timestamp: 1599705049090, author: 50, title: 'Anthony', importance: 5, pageviews: 124371 },
          { id: 3000262223, timestamp: 1599705048872, author: 36, title: 'Mark', importance: 1, pageviews: 13412 },
          { id: 3000262237, timestamp: 1599705048733, author: 61, title: 'Donald', importance: 5, pageviews: 14160 },
          { id: 3000262285, timestamp: 1599705048822, author: 63, title: 'Steven', importance: 3, pageviews: 66912 },
          { id: 3000262327, timestamp: 1599705048844, author: 58, title: 'Paul', importance: 2, pageviews: 107712 },
          { id: 3000262365, timestamp: 1599705048766, author: 21, title: 'Andrew', importance: 1, pageviews: 36756 },
          { id: 3000262381, timestamp: 1599705048646, author: 34, title: 'Joshua', importance: 1, pageviews: 58995 },
          { id: 3000262385, timestamp: 1599705048598, author: 54, title: 'Kenneth', importance: 1, pageviews: 126440 },
          { id: 3000262389, timestamp: 1599705048676, author: 26, title: 'Kevin', importance: 2, pageviews: 200613 },
          { id: 3000262443, timestamp: 1599705049096, author: 49, title: 'Brian', importance: 4, pageviews: 140470 },
          { id: 3000262466, timestamp: 1599705048845, author: 59, title: 'George', importance: 1, pageviews: 96991 },
          { id: 3000262487, timestamp: 1599705048886, author: 21, title: 'Edward', importance: 4, pageviews: 255792 },
          { id: 3000262502, timestamp: 1599705048458, author: 29, title: 'Ronald', importance: 3, pageviews: 131475 },
          { id: 3000262506, timestamp: 1599705048466, author: 64, title: 'Timothy', importance: 2, pageviews: 164736 },
          { id: 3000262519, timestamp: 1599705048586, author: 30, title: 'Jason', importance: 2, pageviews: 84591 },
          { id: 3000262666, timestamp: 1599705048186, author: 24, title: 'Jeffrey', importance: 1, pageviews: 26824 },
          { id: 3000262671, timestamp: 1599705048108, author: 48, title: 'Ryan', importance: 1, pageviews: 152105 },
          { id: 3000262725, timestamp: 1599705048418, author: 43, title: 'Jacob', importance: 1, pageviews: 61680 },
          { id: 3000262762, timestamp: 1599705048566, author: 39, title: 'Gary', importance: 1, pageviews: 64170 },
          { id: 3000262877, timestamp: 1599705048534, author: 31, title: 'Nicholas', importance: 2, pageviews: 272128 },
          { id: 3000262888, timestamp: 1599705048403, author: 62, title: 'Eric', importance: 1, pageviews: 68079 },
          { id: 3000262980, timestamp: 1599705048674, author: 25, title: 'Jonathan', importance: 2, pageviews: 289612 },
          { id: 3000263321, timestamp: 1599705048193, author: 41, title: 'Stephen', importance: 3, pageviews: 109410 },
          { id: 3000263497, timestamp: 1599705048550, author: 44, title: 'Larry', importance: 1, pageviews: 383688 },
          { id: 3000263500, timestamp: 1599705047782, author: 19, title: 'Justin', importance: 1, pageviews: 233914 },
          { id: 3000263504, timestamp: 1599705048380, author: 37, title: 'Scott', importance: 5, pageviews: 404738 },
          { id: 3000263508, timestamp: 1599705048856, author: 30, title: 'Brandon', importance: 3, pageviews: 374751 },
          { id: 3000263536, timestamp: 1599705047878, author: 36, title: 'Benjamin', importance: 5, pageviews: 124760 },
          { id: 3000263549, timestamp: 1599705048000, author: 41, title: 'Samuel', importance: 3, pageviews: 348090 },
          { id: 3000263556, timestamp: 1599705047794, author: 58, title: 'Gregory', importance: 1, pageviews: 401226 },
          { id: 3000263571, timestamp: 1599705047881, author: 55, title: 'Frank', importance: 3, pageviews: 411080 },
          { id: 3000263602, timestamp: 1599705048502, author: 55, title: 'Alexander', importance: 5, pageviews: 280632 },
          { id: 3000263661, timestamp: 1599705048703, author: 24, title: 'Raymond', importance: 5, pageviews: 480555 },
          { id: 3000263667, timestamp: 1599705048224, author: 36, title: 'Patrick', importance: 4, pageviews: 95542 },
          { id: 3000263731, timestamp: 1599705047067, author: 53, title: 'Jack', importance: 4, pageviews: 94987 },
          { id: 3000263742, timestamp: 1599705048022, author: 52, title: 'Dennis', importance: 3, pageviews: 253776 },
          { id: 3000263799, timestamp: 1599705047696, author: 23, title: 'Jerry', importance: 2, pageviews: 100744 },
          { id: 3000263823, timestamp: 1599705047258, author: 22, title: 'Tyler', importance: 4, pageviews: 209100 },
          { id: 3000263838, timestamp: 1599705047212, author: 63, title: 'Aaron', importance: 4, pageviews: 49215 },
          { id: 3000263867, timestamp: 1599705046542, author: 36, title: 'Jose', importance: 1, pageviews: 48724 },
          { id: 3000263906, timestamp: 1599705048657, author: 23, title: 'Adam', importance: 2, pageviews: 452567 },
          { id: 3000264040, timestamp: 1599705048478, author: 48, title: 'Henry', importance: 2, pageviews: 460728 },
          { id: 3000264041, timestamp: 1599705048128, author: 63, title: 'Nathan', importance: 4, pageviews: 468490 },
          { id: 3000264058, timestamp: 1599705047766, author: 44, title: 'Douglas', importance: 2, pageviews: 115920 },
          { id: 3000264073, timestamp: 1599705047677, author: 56, title: 'Zachary', importance: 3, pageviews: 362349 },
          { id: 3000264121, timestamp: 1599705048688, author: 28, title: 'Peter', importance: 4, pageviews: 557728 },
          { id: 3000264137, timestamp: 1599705048201, author: 56, title: 'Kyle', importance: 1, pageviews: 313172 },
          { id: 3000264156, timestamp: 1599705047158, author: 40, title: 'Walter', importance: 4, pageviews: 251760 },
          { id: 3000264209, timestamp: 1599705046508, author: 38, title: 'Ethan', importance: 4, pageviews: 123708 },
          { id: 3000264244, timestamp: 1599705047388, author: 29, title: 'Jeremy', importance: 4, pageviews: 526814 },
          { id: 3000264248, timestamp: 1599705047101, author: 41, title: 'Harold', importance: 4, pageviews: 399609 },
          { id: 3000264250, timestamp: 1599705046614, author: 27, title: 'Keith', importance: 5, pageviews: 267648 },
          { id: 3000264253, timestamp: 1599705046828, author: 53, title: 'Christian', importance: 4, pageviews: 202735 },
          { id: 3000264333, timestamp: 1599705047446, author: 51, title: 'Roger', importance: 3, pageviews: 348942 },
          { id: 3000264413, timestamp: 1599705046476, author: 50, title: 'Noah', importance: 1, pageviews: 64655 },
          { id: 3000264515, timestamp: 1599705047926, author: 24, title: 'Gerald', importance: 4, pageviews: 652460 },
          { id: 3000264529, timestamp: 1599705045694, author: 43, title: 'Carl', importance: 1, pageviews: 138966 },
          { id: 3000264661, timestamp: 1599705047318, author: 20, title: 'Terry', importance: 3, pageviews: 144900 },
          { id: 3000264662, timestamp: 1599705045653, author: 58, title: 'Sean', importance: 5, pageviews: 67024 },
          { id: 3000264689, timestamp: 1599705046678, author: 58, title: 'Austin', importance: 5, pageviews: 70488 },
          { id: 3000264700, timestamp: 1599705047368, author: 35, title: 'Arthur', importance: 5, pageviews: 229731 },
          { id: 3000264722, timestamp: 1599705047338, author: 47, title: 'Lawrence', importance: 3, pageviews: 232878 },
          { id: 3000264724, timestamp: 1599705046558, author: 44, title: 'Jesse', importance: 4, pageviews: 555975 },
          { id: 3000264738, timestamp: 1599705048570, author: 48, title: 'Dylan', importance: 5, pageviews: 812668 },
          { id: 3000264746, timestamp: 1599705046632, author: 57, title: 'Bryan', importance: 5, pageviews: 406021 },
          { id: 3000264759, timestamp: 1599705045190, author: 33, title: 'Joe', importance: 1, pageviews: 157092 },
          { id: 3000264767, timestamp: 1599705046240, author: 57, title: 'Jordan', importance: 4, pageviews: 500544 },
          { id: 3000264778, timestamp: 1599705045878, author: 25, title: 'Billy', importance: 2, pageviews: 77200 },
          { id: 3000264792, timestamp: 1599705046723, author: 24, title: 'Bruce', importance: 4, pageviews: 167103 },
          { id: 3000264798, timestamp: 1599705048000, author: 34, title: 'Albert', importance: 1, pageviews: 875678 },
          { id: 3000264802, timestamp: 1599705045906, author: 59, title: 'Willie', importance: 2, pageviews: 703508 },
          { id: 3000264821, timestamp: 1599705046618, author: 44, title: 'Gabriel', importance: 2, pageviews: 173292 },
          { id: 3000264901, timestamp: 1599705045818, author: 41, title: 'Logan', importance: 5, pageviews: 447015 }
        ]
        // console.log(response.data.total)
        this.total = response.data.total

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false
        }, 1.5 * 1000)
      })
    },
    handleFilter() {
    //   this.listQuery.page = 1
    //   this.getList()
      this.list = [{ id: 3000261852, timestamp: 1599705049516, author: 20, title: 'Robert', importance: 3, pageviews: 3119 }]
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
