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
        this.list = [{ id: 1000000647, timestamp: 1599705049521, author: 18, title: 'Patricia', importance: 1, pageviews: 4217 },
          { id: 1000000676, timestamp: 1599705049504, author: 60, title: 'Jennifer', importance: 3, pageviews: 8462 },
          { id: 1000000695, timestamp: 1599705049468, author: 65, title: 'Linda', importance: 1, pageviews: 15882 },
          { id: 1000000780, timestamp: 1599705049442, author: 51, title: 'Elizabeth', importance: 5, pageviews: 12644 },
          { id: 1000000846, timestamp: 1599705049433, author: 60, title: 'Barbara', importance: 4, pageviews: 26505 },
          { id: 1000000863, timestamp: 1599705049366, author: 38, title: 'Susan', importance: 2, pageviews: 25344 },
          { id: 1000001117, timestamp: 1599705049586, author: 47, title: 'Jessica', importance: 4, pageviews: 52332 },
          { id: 1000001152, timestamp: 1599705049478, author: 56, title: 'Sarah', importance: 2, pageviews: 42576 },
          { id: 1000001194, timestamp: 1599705049270, author: 64, title: 'Karen', importance: 3, pageviews: 38016 },
          { id: 1000001219, timestamp: 1599705049598, author: 64, title: 'Nancy', importance: 5, pageviews: 74760 },
          { id: 1000001288, timestamp: 1599705049184, author: 41, title: 'Lisa', importance: 4, pageviews: 34694 },
          { id: 1000001296, timestamp: 1599705048946, author: 45, title: 'Betty', importance: 3, pageviews: 24756 },
          { id: 1000001313, timestamp: 1599705049480, author: 44, title: 'Margaret', importance: 1, pageviews: 97006 },
          { id: 1000001325, timestamp: 1599705049418, author: 65, title: 'Sandra', importance: 4, pageviews: 74508 },
          { id: 1000001351, timestamp: 1599705048793, author: 32, title: 'Ashley', importance: 1, pageviews: 30945 },
          { id: 1000001524, timestamp: 1599705049622, author: 53, title: 'Kimberly', importance: 5, pageviews: 119616 },
          { id: 1000001538, timestamp: 1599705049184, author: 23, title: 'Emily', importance: 3, pageviews: 72046 },
          { id: 1000001574, timestamp: 1599705048910, author: 34, title: 'Donna', importance: 1, pageviews: 37512 },
          { id: 1000001598, timestamp: 1599705049501, author: 56, title: 'Michelle', importance: 3, pageviews: 121581 },
          { id: 1000001712, timestamp: 1599705048918, author: 28, title: 'Dorothy', importance: 1, pageviews: 84480 },
          { id: 1000001757, timestamp: 1599705048781, author: 60, title: 'Carol', importance: 4, pageviews: 88557 },
          { id: 1000001819, timestamp: 1599705049228, author: 53, title: 'Amanda', importance: 3, pageviews: 116930 },
          { id: 1000001927, timestamp: 1599705048937, author: 34, title: 'Melissa', importance: 3, pageviews: 97313 },
          { id: 1000001928, timestamp: 1599705048982, author: 24, title: 'Deborah', importance: 2, pageviews: 76032 },
          { id: 1000001929, timestamp: 1599705048758, author: 42, title: 'Stephanie', importance: 4, pageviews: 105600 },
          { id: 1000001930, timestamp: 1599705049662, author: 52, title: 'Rebecca', importance: 1, pageviews: 194376 },
          { id: 1000001992, timestamp: 1599705049396, author: 30, title: 'Sharon', importance: 5, pageviews: 201474 },
          { id: 1000002056, timestamp: 1599705049082, author: 42, title: 'Laura', importance: 5, pageviews: 118860 },
          { id: 1000002079, timestamp: 1599705048369, author: 53, title: 'Cynthia', importance: 4, pageviews: 60233 },
          { id: 1000002156, timestamp: 1599705048388, author: 54, title: 'Kathleen', importance: 5, pageviews: 94410 },
          { id: 1000002158, timestamp: 1599705049310, author: 54, title: 'Amy', importance: 4, pageviews: 198152 },
          { id: 1000002161, timestamp: 1599705048470, author: 41, title: 'Shirley', importance: 3, pageviews: 100928 },
          { id: 1000002378, timestamp: 1599705048337, author: 42, title: 'Angela', importance: 1, pageviews: 139161 },
          { id: 1000002385, timestamp: 1599705048164, author: 54, title: 'Helen', importance: 3, pageviews: 70618 },
          { id: 1000002404, timestamp: 1599705048508, author: 54, title: 'Anna', importance: 3, pageviews: 185290 },
          { id: 1000002471, timestamp: 1599705047974, author: 40, title: 'Brenda', importance: 1, pageviews: 113040 },
          { id: 1000002484, timestamp: 1599705049151, author: 29, title: 'Pamela', importance: 2, pageviews: 275835 },
          { id: 1000002525, timestamp: 1599705047810, author: 47, title: 'Nicole', importance: 4, pageviews: 78660 },
          { id: 1000002620, timestamp: 1599705049519, author: 31, title: 'Emma', importance: 4, pageviews: 291291 },
          { id: 1000002743, timestamp: 1599705048078, author: 31, title: 'Samantha', importance: 2, pageviews: 168680 },
          { id: 1000002883, timestamp: 1599705047877, author: 29, title: 'Katherine', importance: 5, pageviews: 85157 },
          { id: 1000002889, timestamp: 1599705048634, author: 20, title: 'Christine', importance: 4, pageviews: 177996 },
          { id: 1000002895, timestamp: 1599705048784, author: 34, title: 'Debra', importance: 1, pageviews: 274254 },
          { id: 1000002910, timestamp: 1599705047842, author: 61, title: 'Rachel', importance: 3, pageviews: 138468 },
          { id: 1000002911, timestamp: 1599705048568, author: 33, title: 'Catherine', importance: 5, pageviews: 190710 },
          { id: 1000002961, timestamp: 1599705049742, author: 46, title: 'Carolyn', importance: 1, pageviews: 343896 },
          { id: 1000003032, timestamp: 1599705048289, author: 24, title: 'Janet', importance: 3, pageviews: 198857 },
          { id: 1000003244, timestamp: 1599705048406, author: 51, title: 'Ruth', importance: 2, pageviews: 152064 },
          { id: 1000003256, timestamp: 1599705048333, author: 34, title: 'Maria', importance: 2, pageviews: 259749 },
          { id: 1000003275, timestamp: 1599705047858, author: 46, title: 'Heather', importance: 2, pageviews: 157700 },
          { id: 1000003278, timestamp: 1599705047671, author: 44, title: 'Diane', importance: 3, pageviews: 215067 },
          { id: 1000003363, timestamp: 1599705047686, author: 47, title: 'Virginia', importance: 5, pageviews: 108368 },
          { id: 1000003451, timestamp: 1599705047120, author: 49, title: 'Julie', importance: 4, pageviews: 109710 },
          { id: 1000003466, timestamp: 1599705048964, author: 47, title: 'Joyce', importance: 5, pageviews: 402570 },
          { id: 1000003476, timestamp: 1599705049228, author: 20, title: 'Victoria', importance: 3, pageviews: 410410 },
          { id: 1000003481, timestamp: 1599705047766, author: 32, title: 'Olivia', importance: 5, pageviews: 236544 },
          { id: 1000004106, timestamp: 1599705047050, author: 32, title: 'Kelly', importance: 5, pageviews: 178980 },
          { id: 1000004122, timestamp: 1599705049210, author: 43, title: 'Christina', importance: 2, pageviews: 432796 },
          { id: 1000004151, timestamp: 1599705049381, author: 42, title: 'Lauren', importance: 4, pageviews: 377541 },
          { id: 1000004170, timestamp: 1599705047218, author: 54, title: 'Joan', importance: 5, pageviews: 188820 },
          { id: 1000004188, timestamp: 1599705049375, author: 40, title: 'Evelyn', importance: 1, pageviews: 390339 },
          { id: 1000004192, timestamp: 1599705047016, author: 56, title: 'Judith', importance: 3, pageviews: 128774 },
          { id: 1000004199, timestamp: 1599705048550, author: 52, title: 'Megan', importance: 1, pageviews: 469224 },
          { id: 1000004233, timestamp: 1599705047382, author: 54, title: 'Cheryl', importance: 4, pageviews: 201856 },
          { id: 1000004235, timestamp: 1599705047998, author: 39, title: 'Andrea', importance: 2, pageviews: 205920 },
          { id: 1000004236, timestamp: 1599705049492, author: 37, title: 'Hannah', importance: 2, pageviews: 492954 },
          { id: 1000004249, timestamp: 1599705046945, author: 21, title: 'Martha', importance: 5, pageviews: 210849 },
          { id: 1000004348, timestamp: 1599705046566, author: 54, title: 'Jacqueline', importance: 5, pageviews: 213520 },
          { id: 1000004353, timestamp: 1599705048385, author: 30, title: 'Frances', importance: 3, pageviews: 292905 },
          { id: 1000004364, timestamp: 1599705048788, author: 23, title: 'Gloria', importance: 2, pageviews: 521850 },
          { id: 1000004369, timestamp: 1599705047925, author: 53, title: 'Ann', importance: 1, pageviews: 452341 },
          { id: 1000004401, timestamp: 1599705047470, author: 50, title: 'Teresa', importance: 2, pageviews: 227592 },
          { id: 1000004429, timestamp: 1599705047952, author: 36, title: 'Kathryn', importance: 1, pageviews: 309374 },
          { id: 1000004430, timestamp: 1599705045784, author: 43, title: 'Sara', importance: 5, pageviews: 152662 },
          { id: 1000004434, timestamp: 1599705049858, author: 32, title: 'Janice', importance: 5, pageviews: 560700 },
          { id: 1000004437, timestamp: 1599705048266, author: 31, title: 'Jean', importance: 3, pageviews: 322620 },
          { id: 1000004438, timestamp: 1599705046401, author: 49, title: 'Alice', importance: 1, pageviews: 159929 },
          { id: 1000004466, timestamp: 1599705049090, author: 58, title: 'Madison', importance: 2, pageviews: 582036 },
          { id: 1000004469, timestamp: 1599705048768, author: 29, title: 'Doris', importance: 5, pageviews: 420438 },
          { id: 1000004482, timestamp: 1599705046678, author: 24, title: 'Abigail', importance: 4, pageviews: 166720 },
          { id: 1000004523, timestamp: 1599705048343, author: 18, title: 'Julia', importance: 2, pageviews: 430515 },
          { id: 1000004538, timestamp: 1599705048492, author: 19, title: 'Judy', importance: 4, pageviews: 523570 },
          { id: 1000004560, timestamp: 1599705048313, author: 23, title: 'Grace', importance: 1, pageviews: 441145 },
          { id: 1000004563, timestamp: 1599705047626, author: 43, title: 'Denise', importance: 3, pageviews: 535164 },
          { id: 1000004568, timestamp: 1599705048623, author: 53, title: 'Amber', importance: 2, pageviews: 633675 }
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
      this.list = [{ id: 1000000647, timestamp: 1599705049521, author: 18, title: 'Patricia', importance: 1, pageviews: 4217 }]
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
