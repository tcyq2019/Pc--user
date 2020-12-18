<template>
  <el-card style="margin-top: 20px">
    <!--
        表单校验：
          1，整体from 表单中
          2，通过prop属性来定义表单名称
          3，定义表单校验的规则
            -再data中定义rules
            绑定给from
          4，校验表单
            -给from 绑定ref
            -this.$ref.supFrom.validate
     -->
    <el-form label-width="80px" :model="spu" :rules="rules" ref="spuForm">
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
      <el-form-item label="SPU描述" prop="description">
        <el-input
          type="textarea"
          placeholder="请输入SPU描述"
          v-model="spu.description"
        ></el-input>
      </el-form-item>
      <el-form-item label="SPU图片" prop="imageList">
        <el-upload
          accept="image/*"
          class="avatar-uploader"
          list-type="picture-card"
          :action="`${$BASE_API}/admin/product/fileUpload`"
          :file-list="formatImageList"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handleAvatarSuccess"
          :before-upload="beforeAvatarUpload"
        >
          <i class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="previewImgUrl" alt="img" />
        </el-dialog>
      </el-form-item>
      <!--
        prop= "saleAttrId"将来用作表单校验
       -->
      <el-form-item label="销售属性" prop="sale">
        <el-select
          :placeholder="`还剩${filterSaleAttrList.length}个未选择`"
          v-model="spu.sale"
        >
          <el-option
            v-for="sale in filterSaleAttrList"
            :label="sale.name"
            :value="`${sale.id}-${sale.name}`"
            :key="sale.id"
          ></el-option>
        </el-select>
        <el-button
          type="primary"
          icon="el-icon-plus"
          @click="addSpuSaleAttr"
          :disabled="!spu.sale"
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
                @close="delTag(i, row)"
                closable
                style="margin-right: 5px"
                v-for="(attrVal, i) in row.spuSaleAttrValueList"
                :key="attrVal.id"
                >{{ attrVal.saleAttrValueName }}</el-tag
              >
              <el-input
                v-if="row.edit"
                size="mini"
                style="width: 100px"
                @blur="editCompleted(row)"
                @keyup.enter.native="editCompleted(row)"
                autofocus
                ref="input"
                v-model="saleAttrValueText"
              >
              </el-input>
              <el-button
                v-else
                icon="el-icon-plus"
                size="mini"
                @click="edit(row)"
                >添加</el-button
              >
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150">
            <template v-slot="{ row, $index }">
              <el-popconfirm
                @onConfirm="delsSpuSaleAttr($index)"
                :title="`确认删除${row.saleAttrName}?`"
              >
                <el-button
                  type="warning"
                  icon="el-icon-delete"
                  size="mini"
                  slot="reference"
                ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="save">保存</el-button>
        <el-button @click="$emit('showList')">取消</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>
<script>
import { mapState } from 'vuex'
import { category } from '@/api'
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
      dialogVisible: false,
      saleAttrValueText: '',
      rules: {
        spuName: [{ required: true, message: '请输入spu名称' }],
        tmId: [{ required: true, message: '请选择品牌' }],
        description: [{ required: true, message: '请输入描述' }],
        imageList: [{ validator: this.imageListValidator, required: true }],
        sale: [{ validator: this.saleValidator, required: true }],
      },
    }
  },
  watch: {
    'category.category3Id'(category3Id) {
      if (!category3Id) return
      this.getAttrList()
    },
    'category.category1Id'() {
      this.clearList()
    },
    'category.category2Id'() {
      this.clearList()
    },
  },
  computed: {
    ...mapState({
      category: (state) => state.category.category,
    }),
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
    //格式化图片数据
    formatImageList() {
      return this.imageList.map((img) => {
        return {
          uid: img.uid || img.id,
          name: img.imgName,
          url: img.imgUrl,
        }
      })
      console.log(this.spu.id)
    },
  },
  methods: {
    save() {
      this.$refs.spuForm.validate(async (valid) => {
        if (valid) {
          console.log('校验通过')
          //发送请求
          const spu = {
            ...this.spu,
            category3Id: this.category.category3Id,
            spuImageList: this.imageList,
            spuSaleAttrList: this.spuSaleAttrList,
          }
          //收集到所有需要的数据 发送请求
          let result
          //判断是否位更新还是修改 因为修改 spu有id
          //发送请求
          if (this.spu.id) {
            result = await this.$API.spu.updateSpu(spu)
          } else {
            result = await this.$API.spu.saveSpu(spu)
          }

          if (result.code === 200) {
            //切换回showList
            this.$emit('showList')
            this.$message.success(`${this.spu.id ? '更新' : '添加'}SPU请求成功`)
          } else {
            this.$message.error(result.message)
          }
        }
      })
    },
    //自定义校验validator
    imageListValidator(rule, value, callback) {
      if (this.imageList.length > 0) {
        callback()
        //校验就通过
        return
      } else {
        callback(new Error('请上传至少一张照片'))
      }
    },
    //判断至少要选择一个销售属性
    saleValidator(rule, value, callback) {
      if (this.spuSaleAttrList.length === 0) {
        callback(new Error('至少选择一个销售属性'))
        return
      }
      const isNotOk = this.spuSaleAttrList.some(
        (sale) => sale.spuSaleAttrValueList.length === 0
      )
      if (isNotOk) {
        callback(new Error('销售属性至少添加一个销售属性值,请添加--'))
        return
      }
      callback()
    },
    //删除整个销售属性
    delsSpuSaleAttr(index) {
      this.spuSaleAttrList.splice(index, 1)
    },
    //删除单个销售属性值
    delTag(index, row) {
      row.spuSaleAttrValueList.splice(index, 1)
    },
    //添加edit 属性给row 用来input的切换的开关
    edit(row) {
      this.$set(row, 'edit', true)
      this.$nextTick(() => {
        this.$refs.input.focus()
      })
    },
    //addSpuSaleAttr 添加销售属性
    addSpuSaleAttr() {
      //选中的属性
      const { sale, id } = this.spu
      const [baseSaleAttrId, saleAttrName] = sale.split('-')
      //直接将销售属性添加到商品中
      this.spuSaleAttrList.push({
        baseSaleAttrId: +baseSaleAttrId, //所有商品的id
        saleAttrName: saleAttrName, //所有销售属性的名称
        spuId: id, //spu id
        spuSaleAttrValueList: [], //销售属性值列表
      })
      //console.log(this.spu);
      this.spu.sale = ''
    },
    //添加销售属性
    editCompleted(row, index) {
      if (this.saleAttrValueText) {
        row.spuSaleAttrValueList.push({
          baseSaleAttrId: row.baseSaleAttrId,
          saleAttrName: row.saleAttrName,
          saleAttrValueName: this.saleAttrValueText,
          spuId: row.spuId,
          uid: row.spuId,
        })
        this.saleAttrValueText = ''
      }
      row.edit = false
    },

    // 上次图片之前触发的回调
    beforeAvatarUpload(file) {
      // console.log(file);
      const imgTypes = ['image/jpg', 'image/png', 'image/jpeg']
      // 检测文件类型
      const isValidType = imgTypes.indexOf(file.type) > -1
      // 检测文件大小
      const isLt = file.size / 1024 < 50

      if (!isValidType) {
        this.$message.error('上传品牌LOGO只能是 JPG 或 PNG 格式!')
      }
      if (!isLt) {
        this.$message.error('上传品牌LOGO大小不能超过 50 kb!')
      }
      // 返回值为true，代表可以上传
      // 返回值为false，代表不可以上传
      return isValidType && isLt
    },
    //上传图片成功回调
    handleAvatarSuccess(res, file) {
      this.imageList.push({
        imgName: file.name, //文件名称
        imgUrl: res.data, //图片地址
        spuId: this.spu.id, //spu id
      })
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
        this.imageList = result.data
      } else {
        this.$message.error(result.message)
        console.log(this.imageList)
      }
    },
    //用element组件里面的预览效果处理图片预览
    handlePictureCardPreview(file) {
      this.previewImgUrl = file.url
      console.log(file)
      this.dialogVisible = true
    },
    //处理图片删除
    handleRemove(file, fileList) {
      //console.log(file, fileList)
      this.imageList = this.imageList.filter((img) => img.imgUrl !== file.url)
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
    this.getSaleAttrList()
    //判断是添加还是修改
    //修改会有id 添加没有id
    if (this.spu.id) {
      this.getSpuImagList()
      this.getSpuSaleAttrList()
    }
  },
}
</script>
<style lang="sass" scoped>
</style>
