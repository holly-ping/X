<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.number"
        placeholder="门票名称"
        clearable
        class="filter-item el-form-default-width"
      />
      <el-select
        v-model="listQuery.enable"
        placeholder="门票状态"
        clearable
        class="filter-item el-form-default-width"
      >
        <el-option
          v-for="(item, index) in [{value:0,label:'下架'},{value:1,label:'上架'},{value:2,label:'全部'}]"
          :key="index"
          :label="item.label"
          :value="item.value"
        />
      </el-select>
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="clickSearch">搜索</el-button>
    </div>
    <div class="filter-container right">
      <!-- <el-button type="primary" icon="el-icon-arrow-right" @click="clickClassification">商品分类</el-button> -->
      <el-button type="primary" icon="el-icon-plus" @click="addIfication">新增</el-button>
    </div>
    <el-table
      v-loading="listLoading"
      :data="table"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="商品编号">
        <template slot-scope="scope">{{ scope.row.number }}</template>
      </el-table-column>
      <el-table-column align="center" label="商品名称">
        <template slot-scope="scope">{{ scope.row.name }}</template>
      </el-table-column>
      <el-table-column align="center" label="有效期">
        <template slot-scope="scope">{{ scope.row.categoryFirstName }}</template>
      </el-table-column>
      <el-table-column align="center" label="价格">
        <template slot-scope="scope">{{ scope.row.categoryTwoName }}</template>
      </el-table-column>
      <el-table-column align="center" label="排序">
        <template slot-scope="scope">{{ scope.row.price }}</template>
      </el-table-column>
      <el-table-column align="center" label="可邀请人">
        <template slot-scope="scope">{{ scope.row.seq }}</template>
      </el-table-column>
      <el-table-column align="center" label="商品状态">
        <template slot-scope="scope">
          <el-button
            :type="scope.row.enable ? 'primary' : ''"
            size="mini"
            @click="handelClick(scope.row)"
          >{{ scope.row.enable==1 ? '上架' : '下架' }}</el-button>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="300" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="clickWatch(scope.row)">修改</el-button>
          <el-button type="danger" size="mini" @click="clickDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="pagination-container">
      <el-pagination
        v-show="page.total>0"
        :current-page="listQuery.current"
        :total="page.total"
        :page-size="listQuery.size"
        :page-sizes="page.sizes"
        background
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="changePageSize"
        @current-change="clickPageCurrent"
      />
    </div>
  </div>
</template>
<script>
import { orderingGoodsList, deleteGoods } from '../../../api/commodity'
import {
  sysOrderingGoodsCategoryFirstList,
  orderingGoodsSetEnable
} from '../../../api/commodity'
export default {
  name: 'RepastProductsTable',
  data() {
    return {
      form: {
        type: '',
        typeList: [],
        number: '',
        status: '',
        statusList: [],
        name: '',
        rules: []
      },

      table: [],
      listQuery: {
        current: 1,
        size: 20,
        name: null,
        categoryId: null,
        number: null,
        enable: 2
      },
      listLoading: {
        table: false
      },
      page: {
        total: 0,
        sizes: [10, 20, 30, 50]
      }
    }
  },
  created() {
    // this.getFristList()
    this.getList()
  },
  mounted() {},
  methods: {
    // clickClassification() {
    //   this.$router.push('/commodity/drinkclass')
    // },
    getList() {
      this.listLoading = true
      orderingGoodsList(this.listQuery).then(response => {
        this.table = response.data.records
        this.page.total = parseInt(response.data.total)
        this.listLoading = false
      })
    },
    getFristList() {
      sysOrderingGoodsCategoryFirstList().then(response => {
        this.form.typeList = response.data
      })
    },
    /**
     * 商品上架
     */
    clickPutaway() {},

    clickWatch(data) {
      this.$router.push({
        path: '/secKill/entranceDetail',
        query: { id: data.id }
      })
      // ?id:' + data.id
    },
    clickDelete(row) {
      if (row.enable == 0) {
        const funDelete = () =>
          deleteGoods(row.id).then(res => {
            this.$message({
              message: '删除成功',
              type: 'success'
            })
            this.getList()
          })
        this.$confirm('确认删除？').then(_ => {
          funDelete()
        })
      } else {
        this.$confirm('商品上架状态不可删除', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'success',
          center: true
        }).then(() => {})
      }
    },

    clickPageCurrent(val) {
      this.listQuery.current = val
      this.getList()
    },
    changePageSize(val) {
      this.listQuery.size = val
      this.getList()
    },
    clickSearch() {
      this.listQuery.current = 1
      this.getList()
    },
    addIfication() {
      this.$router.push({ name: 'MessagePushEditor' })
    },
    handelClick(val) {
      orderingGoodsSetEnable({
        id: val.id,
        enable: !val.enable
      })
        .then(res => {
          // 上架下架
          this.$message({
            message: '商品状态成功',
            type: 'success'
          })
          this.getList()
        })
        .catch(res => {})
    }
  }
}
</script>

<style rel="stylesheet/scss" scoped lang="scss">
.el-form-default-width {
  width: 200px;
}
.right {
  text-align: right;
}

.filter-container {
  padding-bottom: 10px;
  vertical-align: middle;
}
</style>
