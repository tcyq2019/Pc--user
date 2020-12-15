<template>
  <el-card>
    <el-form :inline="true" :model="category">
      <el-form-item label="一级列表">
        <el-select
          v-model="category.category1Id"
          placeholder="请选择"
          @change="handleSelectChange1"
          :disabled="disabled"
        >
          <el-option
            v-for="c1 in category1List"
            :key="c1.id"
            :label="c1.name"
            :value="c1.id"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级列表">
        <el-select
          v-model="category.category2Id"
          placeholder="请选择"
          @change="handleSelectChange2"
        >
          <el-option
            v-for="c2 in category2List"
            :key="c2.id"
            :label="c2.name"
            :value="c2.id"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级列表">
        <el-select
          @change="handleSelectChange3"
          v-model="category.category3Id"
          placeholder="请选择"
        >
          <el-option
            v-for="c3 in category3List"
            :key="c3.id"
            :label="c3.name"
            :value="c3.id"
          >
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
  </el-card>
</template>
<script>
export default {
  name: 'category',
  props:['disabled','clearList '],
  data() {
    return {
      category: {
        category1Id: '' /* 1级分类id */,
        category2Id: '',
        category3Id: '',
      },
      category1List: [] /* 一级分类数据 */,
      category2List: [],
      category3List: [],
    }
  },
  methods: {
    async handleSelectChange1(category1Id) {
      ;(this.category2List = []), (this.category3List = [])
      this.category.category2Id = ''
      this.category.category3Id = ''
      const result = await this.$API.attrs.getCategorys2(category1Id)
      if (result.code === 200) {
        this.category2List = result.data
      } else {
        this.$message.error(result.message)
      }
       this.$bus.$emit("clearList");
    },
    async handleSelectChange2(category2Id) {
      this.category3List = []
      this.category.category3Id = ''
      const result = await this.$API.attrs.getCategorys3(category2Id)
      if (result.code === 200) {
        this.category3List = result.data
      } else {
        this.$message.error(result.message)
      }
      this.$bus.$emit("clearList");
    },
    async handleSelectChange3(category3Id) {
      const category = {
        ...this.category,
        category3Id,
      }
       this.$bus.$emit('change',category)

    },
  },
  async mounted() {
    const result = await this.$API.attrs.getCategorys1()
    if (result.code === 200) {
      this.category1List = result.data
    } else {
      this.$message.error(result.message)
    }
  },
}
</script>
<style lang="sass" scoped>
</style>
