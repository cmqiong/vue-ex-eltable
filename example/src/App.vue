<template>
  <div class="app-container">
    <!--搜索框-->
    <div class="search">
      <b>状态：</b>
      <el-select v-model="status" placeholder="请选择状态" no-data-text="暂无数据，请等待加载">
        <el-option label="ALL" value="all"></el-option>
        <el-option label="OPEN" value="open"></el-option>
        <el-option label="CLOSED" value="closed"></el-option>
      </el-select>

      <el-button style="margin-top: 10px" type="primary" plain icon="el-icon-search" @click="submitSearch()">
        搜索
      </el-button>
    </div>

    <ex-table
      ref="table1"
      v-loading="listLoading"
      element-loading-text="加载中，请稍后..."
      :data="data"
      :search-method="fetchRemoteData"
      :page-size="3"
      :page-sizes="[3, 5, 10]">
      <el-table-column label="ID" prop="id"/>
      <el-table-column label="NUMBER" prop="number"/>
      <el-table-column
        prop="state"
        label="STATE"
        :filters="[{ text: 'open', value: 'OPEN' }, { text: 'closed', value: 'CLOSED' }]"
        :filter-method="filterTag"
        filter-placement="bottom-end">
        <template slot-scope="scope">
          <el-tag :type="scope.row.state === 'open' ? 'warning' : 'success'"
                  disable-transitions>{{scope.row.state}}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="TITLE" prop="title"/>
    </ex-table>

    <!-- <el-button @click="$refs.table1.setCurrentPage(1)">回到首页</el-button> -->
  </div>
</template>
<script>
  import ExTable from './vue-ex-eltable/index.js'
  import axios   from 'axios'

  export default {
    components: {ExTable},
    data() {
      return {
        listLoading: false,
        status: '',
        lastSearchParameters: {},
        data: []
      }
    },
    methods: {
      // 在此操作对查询参数进行转换
      updateLastSearchParams() {
        this.handleSearchParams('state', this.status)
      },
      handleSearchParams(queryParamName, sourceParam) {
        if (!sourceParam.length) {
          delete(this.lastSearchParameters[queryParamName])
        } else {
          this.lastSearchParameters[queryParamName] = sourceParam
        }
      },
      // 防止不同接口分页参数不同
      updateLastSearchPages(currentPage, pageSize) {
        // 此处对请求页码参数进行转换
        currentPage && (this.lastSearchParameters.page = currentPage)
        pageSize && (this.lastSearchParameters.per_page = pageSize)
      },
      // 数据获取渲染
      fetchRemoteData(currentPage, pageSize) {
        if (this.listLoading) return

        this.listLoading = true
        this.updateLastSearchPages(currentPage, pageSize)
        console.log('lastSearchParameters', this.lastSearchParameters)

        axios.get('https://api.github.com/repos/vmg/redcarpet/issues', {
          params: this.lastSearchParameters
        }).then(response => {
          this.data = response.data
          this.$refs.table1.pagination.total = 50
          this.listLoading = false
        }).catch(err => {
          this.$message.error('请求失败，请重试')
          this.data = []
          this.$refs.table1.pagination.total = 0
          this.listLoading = false
        })
      },
      submitSearch() {
        this.updateLastSearchParams()
        this.fetchRemoteData(1)
      },
      filterTag(value, row) {
        return row.tag === value
      }
    },
    mounted() {
      this.$refs.table1.fetchData()
    }
  }
</script>
<style scoped>
  .app-container {
    margin: 20px;
  }
</style>

