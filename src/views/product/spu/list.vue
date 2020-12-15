<template>
<div>
  <Category />
  <SpuList v-if="isShowList" @showUpdateList="showUpdateList"/>
  <SpuUpdataList v-else :item="item" />
</div>
</template>

<script>
import Category from "@/components/categoryList"
import SpuList from "./spuList"
import SpuUpdataList from "./spuUpdataList"
export default {
  name: 'list',
  data(){
    return{
      isShowList:true,
      item:{},
    }
  },
  methods:{
    showUpdateList(row){
      this.isShowList = false;
      this.item={...row}
    }
  },
  components:{
    Category,
    SpuList,
    SpuUpdataList
  }
}
</script>
