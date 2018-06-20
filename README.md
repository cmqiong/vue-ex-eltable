## 写在前面
感谢本程序主要代码来源 [Rudolph的博客](https://w-rudolph.github.io/2017/11/04/element-ui-extend-table/)

## vue-ex-eltable
> 结合 element-ui form + pagination，表格渲染自带分页功能，可控制是否开启该功能，并整合了前端业务代码示例。

## Notices
使用条件：
- 使用 elementUI

## Installation
```shell
npm i vue-ex-eltable
```

## Usage
详细代码参见 example/src/APP.vue
.vue
```html
<template>
  <div class="app-container">
    <ex-table
      ref="table1"
      v-loading="listLoading"
      element-loading-text="加载中，请稍后..."
      :data="data"
      :search-method="fetchRemoteData"
      :page-size="3"
      :page-sizes="[3, 5, 10]">
      <el-table-column label="ID" prop="id"/>
    </ex-table>
  </div>
</template>
<script>
import ExTable from './vue-ex-eltable/index.js'

export default {
    components: {ExTable},
    data() {
      return {
        listLoading: false,
        lastSearchParameters: {},
        data: []
      }
    },
    methods: {
      // 在此操作对查询参数进行转换
      updateLastSearchParams() {
        this.handleSearchParams('state', this.status)
      },
      // 如果该查询参数为空字符串或无，则删掉该参数
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

        // 获取数据操作...
        this.data = []
      },
      submitSearch() {
        this.updateLastSearchParams()
        this.fetchRemoteData(1)
      }
    },
    mounted() {
      this.$refs.table1.fetchData()
    }
  }
</script>
```

### Attribute
- 概述：ELTable 的参数一样，原 Pagination 参数写到 ExTable 上，个别参数有修改，详见下面。

  - [ELTable 参数传送门](https://element.faas.ele.me/#/zh-CN/component/table)
  - [Pagination 参数传送门](https://element.faas.ele.me/#/zh-CN/component/pagination)

- 本插件有变化的 pagination 属性：

| 参数     | 说明          | 类型         | 可选值           | 默认值     |
| -------- | ------------- | ----------- | --------------- | --------- |
| searchMethod | 必选参数，获取数据的操作 | Function | - | ()=>{} |
| show-pagination | 可选参数，是否显示分页组件 | Boolean | true、false | true |
| pagination-layout | 可选参数，组件布局，子组件名用逗号分隔 | String | - | total, sizes, prev, pager, next, jumper |
| page-size | 可选参数，每页显示条目个数 | Number | - | 10 |
| page-sizes | 可选参数，每页显示个数选择器的选项设置 | Number[] | - | [10, 25, 50, 100] |
