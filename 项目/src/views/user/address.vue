<template>
  <div class="app-container">

    <!-- 查询和其他操作 -->
    <div class="filter-container">
      <el-input v-model="listQuery.userId" clearable class="filter-item" style="width: 200px;" placeholder="请输入用户ID"/>
      <el-input v-model="listQuery.name" clearable class="filter-item" style="width: 200px;" placeholder="请输入收货人名称"/>
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">查找</el-button>
      <el-button :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload">导出</el-button>
    </div>

    <!-- 查询结果 -->
    <el-table v-loading="listLoading" :data="list" element-loading-text="正在查询中。。。" border fit highlight-current-row>

      <el-table-column align="center" label="地址ID" prop="id" width="100px" sortable/>

      <el-table-column align="center" label="用户ID" prop="userId" width="150px"/>

      <el-table-column align="center" label="收货人名称" prop="name" width="200px"/>

      <el-table-column align="center" label="手机号码" prop="tel" width="200px"/>

      <el-table-column align="center" label="区域地址" prop="province"/>

      <el-table-column align="center" label="详细地址" prop="addressDetail"/>

      <el-table-column :data="tableData" align="center" label="默认" prop="isDefault" width="100px" :formatter="statusFormatter"/>

    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

  </div>
</template>

<script>
import { listAddress } from '@/api/user'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination

export default {
  name: 'Address',
  components: { Pagination },
  data() {
    return {
      tableData:[],
      list: [],
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        username: undefined,
        sort: 'add_time',
        order: 'desc'
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
      listAddress(this.listQuery).then(response => {
        console.log(response)
        for(let i =0;i<response.data.data.list.length;i++){
          response.data.data.list[i].province=response.data.data.list[i].province + response.data.data.list[i].city + response.data.data.list[i].county
        }
        this.list = response.data.data.list
        this.total = response.data.data.total
        this.listLoading = false
      }).catch(() => {
        this.list = []
        this.total = 0
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['地址ID', '用户ID', '收货人名称', '区域地址', '详细地址']
        const filterVal = ['id', 'username', 'content', 'picUrls', 'addTime']
        excel.export_json_to_excel2(tHeader, this.list, filterVal, '意见反馈信息')
        this.downloadLoading = false
      })
    },
    statusFormatter(row, column){
        const isDefault=row.isDefault
        if(isDefault==0){
          return'否'
        }else if(isDefault==1){
          return'是'
        }else{
          return
        }
      },
  }
}
</script>
