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
import { mapState, mapActions, mapMutations } from 'vuex'
export default {
  name: 'category',
  props: ['disabled', 'clearList '],
  data() {
    return {
      category: {
        category1Id: '' /* 1级分类id */,
        category2Id: '',
        category3Id: '',
      },
    }
  },
  computed: {
    ...mapState({
      category1List: (state) => state.category.category1List,
      category2List: (state) => state.category.category2List,
      category3List: (state) => state.category.category3List,
    }),
  },

  methods: {
    ...mapMutations(['category/SET_CATEGORY3_ID']),
    ...mapActions([
      'category/getCategory1List',
      'category/getCategory2List',
      'category/getCategory3List',
    ]),
    async handleSelectChange1(category1Id) {
      this.category.category2Id = ''
      this.category.category3Id = ''
      this['category/getCategory2List'](category1Id)
      /* this.$bus.$emit("clearList"); */
    },
    async handleSelectChange2(category2Id) {
      this.category.category3Id = ''

      this['category/getCategory3List']({ category2Id })
      //this.$bus.$emit("clearList");
    },
    async handleSelectChange3(category3Id) {
      this['category/SET_CATEGORY3_ID'](category3Id)
    },
  },
  async mounted() {
    this['category/getCategory1List']()
  },
}
</script>
<style lang="sass" scoped>
</style>
