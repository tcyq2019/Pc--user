<template>
  <el-card style="margin-top: 20px">
    <el-button type="primary" icon="el-icon-plus">添加SPU</el-button>

    <el-table :data="spuList" border style="width: 100%; margin: 20px 0">
      <el-table-column type="index" label="序号" width="80" align="center">
      </el-table-column>
      <el-table-column prop="spuName" label="SPU名称">
      </el-table-column>
      <el-table-column prop="description" label="SPU描述"> </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{row}">
          <el-button type="primary" icon="el-icon-plus" size="mini"></el-button>
          <el-button type="primary"
           icon="el-icon-edit"
            size="mini"
            @click="$emit('showUpdateList',row)"
            ></el-button>
          <el-button type="info" icon="el-icon-edit" size="mini"></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      class="pagination"
      @size-change="getPageList(page, $event)"
      @current-change="getPageList($event, limit)"
      :page-sizes="[3, 6, 9]"
      :page-size.sync="limit"
      :current-page="page"
      layout="prev, pager, next, jumper, sizes, total"
      :total="total"
    >
    </el-pagination>
  </el-card>
</template>
<script>
export default {
  name: 'spuList',
  data() {
    return {
      page: 1,
      limit: 3,
      total: 0,
      category: {
        category1Id: '',
        category2Id: '',
        category3Id: '',
      },
      spuList: [],
      loading: false,
    }
  },
  methods: {
    //获取SPU分页列表
    async getPageList(page, limit) {
      const { category3Id } = this.category
      const result = await this.$API.spu.getSpuList({
        page,
        limit,
        category3Id,
      })
      if (result.code === 200) {
        this.$message.success('获取SPU分页列表成功')
        console.log(1111)
        console.log(result.data.records)
        this.spuList = result.data.records
        this.total = result.data.total
      } else {
        this.$message.error(result.message)
      }
    },
    //处理category的change（当选中三级分类时触发）
    handleCategoryChange(category) {
      //触发事件会将分类id传过来
      this.category = category
      this.getPageList(this.page, this.limit)
    },
    clearList() {
      //重新再选中一级或者二级的时候 清空
      this.spuList = []
      this.page = 1
      this.limit = 3
      this.total = 0
      this.category.category3Id = ''
    },
  },
  mounted() {
    this.$bus.$on('change', this.handleCategoryChange)
    this.$bus.$on('clearList', this.clearList)
  },
  beforeDestroy(){
     this.$bus.$off('change', this.handleCategoryChange)
    this.$bus.$off('clearList', this.clearList)
  }
}
</script>
<style lang="sass" scoped>
</style>
