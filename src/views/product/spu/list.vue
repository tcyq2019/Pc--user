<template>
  <div>
    <SkuList v-if="isShowSkuList" :skuItem="skuItem" />
    <div v-else>
      <Category :disabled="!isShowList" />
      <SpuList
        v-if="isShowList"
        @showUpdateList="showUpdateList"
        @showSpuList="showSpuList"
      />
      <SpuUpdataList v-else :item="item" @showList="showList" />
    </div>
  </div>
</template>

<script>
import Category from '@/components/categoryList'
import SpuList from './spuList'
import SpuUpdataList from './spuUpdataList'
import SkuList from './skuList'
export default {
  name: 'list',
  data() {
    return {
      isShowList: true,
      item: {},
      isShowSkuList: false,
      skuItem: {},
    }
  },
  methods: {
    showSpuList(row) {
      this.isShowSkuList = true
      this.skuItem = { ...row }
    },
    showUpdateList(row) {
      this.isShowList = false
      this.item = { ...row }
      console.log(row)
    },
    showList(category3Id) {
      this.isShowList = true
     /*  this.$nextTick(() => {
        this.$bus.$emit('change', { category3Id })
      }) */
    },
  },
  components: {
    Category,
    SpuList,
    SpuUpdataList,
    SkuList,
  },
}
</script>
