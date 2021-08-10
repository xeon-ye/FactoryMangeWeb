<template>
  <a-card :bordered="false">
    <div class="table-operator">
      <a-button type="primary" icon="plus" @click="hanldleAdd()">新建</a-button>
      <!-- <a-button type="primary" icon="minus" @click="handleDelete(selectedRowKeys)" :disabled="!hasSelected()" :loading="loading">删除</a-button> -->
      <a-button type="primary" icon="redo" @click="getDataList()">刷新</a-button>
    </div>

    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="10">
          <a-col :md="4" :sm="24">
            <a-form-item label="查询类别">
              <a-select allowClear v-model="queryParam.condition">
                <a-select-option key="StyleId">款式ID</a-select-option>
                <a-select-option key="StyleNo">款号</a-select-option>
                <a-select-option key="ClientStyleNo">客户款号</a-select-option>
                <a-select-option key="ClientName">客户</a-select-option>
                <!-- <a-select-option key="SeachStr">模糊查找</a-select-option>s -->
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="4" :sm="24">
            <a-form-item>
              <a-input v-model="queryParam.keyword" placeholder="关键字" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="24">
            <a-button type="primary" @click="() => {this.pagination.current = 1; this.getDataList()}">查询</a-button>
            <a-button style="margin-left: 8px" @click="() => (queryParam = {})">重置</a-button>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <a-table ref="table" :columns="columns" :rowKey="row => row.Id" :dataSource="data" :pagination="pagination" :loading="loading" @change="handleTableChange"
      :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }" :bordered="true" size="small">
      <span slot="action" slot-scope="text, record">
        <template>
          <a @click="handleEdit(record.Id)">查看</a>
          <a-divider type="vertical" />
          <a @click="handleDelete([record.Id])">删除</a>
        </template>
      </span>

      <div slot="pic" slot-scope="text,record">
        <a href="#">
          <img slot=" cover" alt="example" v-if="record.PicUrl" class="img2" :src="'http://localhost:5000'+record.PicUrl" @click="handleEdit(record.Id)" />
          <img slot="cover" alt="example" v-else class="img2" @click="handleEdit(record.Id)" src="../../../assets/nopic.jpg" />
        </a>

      </div>
    </a-table>

    <edit-form ref="editForm" :parentObj="this" @change="getDataList"></edit-form>
  </a-card>
</template>

<script>
import EditForm from './StyleEditForm'
// import nopic from '../../../assets/'
const nopic = "../../../assets/nopic.jpg";
const columns = [
  { title: '款式图', dataIndex: 'PicUrl', width: '60px', scopedSlots: { customRender: 'pic' }, },

  { title: '款式ID', dataIndex: 'StyleId', width: '10%' },
  { title: '款号', dataIndex: 'StyleNo', width: '10%' },
  { title: '客户', dataIndex: 'ClientName', width: '10%' },
  { title: '品牌', dataIndex: 'Brand', width: '10%' },
  { title: '克重(g)', dataIndex: 'Weight', width: '6%' },
  { title: '录入日期', dataIndex: 'CreateTime', width: '10%' },

  // { title: '针型', dataIndex: 'Gauge', width: '10%' },
  { title: '操作', dataIndex: 'action', scopedSlots: { customRender: 'action' } }
]

export default {
  components: {
    EditForm
  },
  mounted() {
    this.getDataList()
  },
  data() {
    return {
      data: [],
      pagination: {
        current: 1,
        pageSize: 10,
        showTotal: (total, range) => `总数:${total} 当前:${range[0]}-${range[1]}`
      },
      filters: {},
      sorter: { field: 'Id', order: 'asc' },
      loading: false,
      columns,
      queryParam: {},
      selectedRowKeys: []
    }
  },
  methods: {
    handleTableChange(pagination, filters, sorter) {
      this.pagination = { ...pagination }
      this.filters = { ...filters }
      this.sorter = { ...sorter }
      this.getDataList()
    },
    getDataList() {
      this.selectedRowKeys = []

      this.loading = true
      this.$http
        .post('/Prod/Prod_Style/GetDataList', {
          PageIndex: this.pagination.current,
          PageRows: this.pagination.pageSize,
          SortField: this.sorter.field || 'Id',
          SortType: this.sorter.order,
          Search: this.queryParam,
          ...this.filters
        })
        .then(resJson => {
          this.loading = false
          this.data = resJson.Data
          const pagination = { ...this.pagination }
          pagination.total = resJson.Total
          this.pagination = pagination
          console.log(this.data)
        
        })
    },
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    hasSelected() {
      return this.selectedRowKeys.length > 0
    },
    hanldleAdd() {
      this.$refs.editForm.newStyle()
    },
    handleEdit(id) {
      this.$refs.editForm.loadStyle(id)
    },
    handleDelete(ids) {
      var thisObj = this
      this.$confirm({
        title: '确认删除吗?',
        onOk() {
          return new Promise((resolve, reject) => {
            thisObj.$http.post('/Prod/Prod_Style/DeleteData', ids).then(resJson => {
              resolve()

              if (resJson.Success) {
                thisObj.$message.success('操作成功!')

                thisObj.getDataList()
              } else {
                thisObj.$message.error(resJson.Msg)
              }
            })
          })
        }
      })
    }
  }
}
</script>
<style scoped>
.img2 {
  width: 60px;
  height: 80px;
  object-fit: cover;
}
</style>