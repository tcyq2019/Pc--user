<template>
  <el-card style="margin-top: 20px">
    <el-form label-width="80px" :model="spu">
      <el-form-item label="SPU名称" prop="spuName">
        <el-input placeholder="请输入SPU名称" v-model="spu.spuName"></el-input>
      </el-form-item>
      <el-form-item label="品牌" prop="tmId">
        <el-select placeholder="请选择品牌" v-model="spu.tmId">
          <el-option
            v-for="trade in trademarkList"
            :key="trade.id"
            :label="trade.tmName"
            :value="trade.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述">
        <el-input
          type="textarea"
          placeholder="请输入SPU描述"
          v-model="spu.description"
        ></el-input>
      </el-form-item>
      <el-form-item label="SPU图片">
        <!-- <el-upload
          action="https://jsonplaceholder.typicode.com/posts/"
        list-type="picture-card"
        :on-preview="handlePictureCardPreview"
        :on-remove="handleRemove">
        <i class="el-icon-plus"></i>
          </el-upload> -->
        <el-upload
          class="avatar-uploader"
          list-type="picture-card"
          :action="`${$BASE_API}/admin/product/fileUpload`"
          :file-list="imageList"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
        >
          <i class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
        <span>只能上传jpg/png文件，且不超过50kb</span>
      </el-form-item>
      <!--
        prop= "saleAttrId"将来用作表单校验
       -->
      <el-form-item label="销售属性" prop="saleAttrId">
        <el-select
          :placeholder="`还剩${filterSaleAttrList.length}个未选择`"
          v-model="spu.saleAttrId"
        >
          <el-option
            v-for="sale in filterSaleAttrList"
            :label="sale.name"
            :value="sale.id"
            :key="sale.id"
          ></el-option>
        </el-select>
        <el-button type="primary" icon="el-icon-plus" @click="addvalue"
          >添加销售属性</el-button
        >
        <el-table
          :data="spuSaleAttrList"
          border
          style="width: 100%; margin: 20px 0"
        >
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="saleAttrName" label="属性名称" width="150">
          </el-table-column>

          <el-table-column label="属性值列表">
            <template v-slot="{ row }">
              <el-tag
                style="margin-right: 5px"
                v-for="attrVal in row.spuSaleAttrValueList"
                :key="attrVal.id"
                >{{ attrVal.saleAttrValueName }}</el-tag
              >
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150">
            <template>
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
              ></el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary">保存</el-button>
        <el-button>取消</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>
<script>
import { set } from 'nprogress'
export default {
  name: 'spuUpdata',
  props: {
    item: Object,
  },
  data() {
    return {
      spu: this.item,
      trademarkList: [], //所有品牌数据
      imageList: [], //所有图片列表
      previewImgUrl: '', //图片地址
      visible: false, //图片对话框的显示或隐藏
      saleAttrList: [], //所有销售列表
      spuSaleAttrList: [], //当前SPU销售属性列表
    }
  },
  computed: {
    //这里将所有销售列表和spu销售列表进行筛选被spu销售列表选中的就会被所有销售列表
    //剔除

    filterSaleAttrList() {
      return this.saleAttrList.filter((sale) => {
        //this.spuSaleAttrList.indexOf()只适用于数组中的基本类型
        //而find 适用于数组中的引用类型 找到了返回{} 没找到就返回去一个undefined
        return !this.spuSaleAttrList.find(
          (spuSale) => spuSale.baseSaleAttrId === sale.id
        )
      })
    },
  },
  methods: {
    //试写添加属性
    addvalue() {
      //this.$set(row, 'edit', true)
      let setIdspu = this.spu.saleAttrId
      const setId = this.saleAttrList.find((sale) => {
        return sale.id === setIdspu
      })
      console.log(setId)
     /*  this.$set(this.spuSaleAttrList, 'setId',{
        id:setId.id,
        name:setId.name
      }) */
      this.spuSaleAttrList.push(setId)
    },
    //重上之下写 首先遇到的第一个请求  获取所有的品牌数据
    //请求成功后把数据给到自己定义的trademarkList
    async getTrademarkList() {
      const result = await this.$API.spu.getTrademarkList()
      if (result.code === 200) {
        this.$message.success('获取所有品牌成功')
        /* console.log(result.data); */
        this.trademarkList = result.data
      } else {
        this.$message.error(result.message)
      }
    },
    //获取所有的图片数据
    async getSpuImagList() {
      const { id } = this.spu
      const result = await this.$API.spu.getSpuImageList(id)
      if (result.code === 200) {
        //console.log(item)
        this.$message.success('获取所有图片成功')
        this.imageList = result.data.map((img) => {
          return {
            id: img.id,
            name: img.imgName,
            url: img.imgUrl,
          }
        })
      } else {
        this.$message.error(result.message)
      }
    },
    //用element组件里面的预览效果处理图片预览
    handlePictureCardPreview(file) {
      this.previewImgUrl = file.url
      this.visible = true
    },
    //处理图片删除
    handleRemove(file, fileList) {
      //console.log(file, fileList)
      this.imageList = this.imageList.filter((img) => img.id !== file.id)
    },
    //获取所有的销售列表
    async getSaleAttrList() {
      const result = await this.$API.spu.getSaleAttrList()
      if (result.code === 200) {
        this.$message.success('获取所有的销售属性成功')
        //处理数据
        this.saleAttrList = result.data
      } else {
        this.$message.error(result.message)
      }
    },
    //请求spu销售列表进行渲染
    async getSpuSaleAttrList() {
      const { id } = this.spu
      const result = await this.$API.spu.getSpuSaleAttrList(id)
      if (result.code === 200) {
        this.$message.success('获取属性成功')
        //处理数据
        //console.log(result.data)
        this.spuSaleAttrList = result.data
      } else {
        this.$message.error(result.message)
      }
    },
  },
  async mounted() {
    this.getTrademarkList()
    this.getSpuImagList()
    this.getSaleAttrList()
    this.getSpuSaleAttrList()
  },
}
</script>
<style lang="sass" scoped>
</style>
