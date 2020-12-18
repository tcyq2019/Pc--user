<template>
  <div>
    <el-button type="primary" icon="el-icon-plus" @click="add">添加 </el-button>
    <el-table border style="width: 100%; margin: 20px 0" :data="tradeAllList">
      <el-table-column type="index" label="序号" width="80" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称"></el-table-column>
      <el-table-column label="品牌LOGO">
        <template v-slot="scope">
          <!-- {{ JSON.stringify(scope) }} -->
          <!--
            scope代表所有数据
              scope.row 代表当前行所有数据
          -->
          <img class="trademark-img" :src="scope.row.logoUrl" alt="logo" />
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot="{ row }">
          <el-button type="waring" icon="el-icon-edit" @click="update(row)">
            修改
          </el-button>
          <el-button type="danger" icon="el-icon-delete" @click="del(row)"> 删除 </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- <el-pagination
      class="trademark-pagination"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :page-sizes="[3, 6, 9]"
      :page-size.sync="limit"
      :current-page="page"
      layout="prev, pager, next, jumper, sizes, total"
      :total="total"
    >
    </el-pagination> -->

    <!--
      :page-size.sync="limit"   可以让limit更新变成同步更新
      :current-page.sync="page" 可以让page更新变成同步更新

      $event
        1.  在DOM事件中代表 event
        <button @click="handle(123, $event)"></button>
        触发事件是浏览器的行为，所以$event代表event

        2. 在自定义事件中代表 第一个参数
          <button @aaa="handle($event)"></button>
          假设这样触发自定义事件： this.$emit('aaa', 123, 456);
          那么$event就为123（第一个参数）
    -->
    <el-pagination
      @size-change="getpageList(page, $event)"
      @current-change="getpageList($event, limit)"
      class="pageination"
      :page-sizes="[3, 6, 9]"
      :page-size.sync="limit"
      :current-page.sync="page"
      layout="prev,pager,next,jumper,sizes,total"
      :total="total"
    >
    </el-pagination>
    <el-dialog
      :title="`${trademarkForm.id ? '修改' : '添加'}品牌`"
      :visible.sync="visible"
      width="50%"
    >
      <el-form
        :model="trademarkForm"
        :rules="rules"
        ref="trademarkForm"
        label-width="100px"
      >
        <el-form-item label="品牌名称" prop="tmName">
          <el-input v-model="trademarkForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌LOGO" prop="logoUrl">
          <!--
            前提允许跨域
              action="http://182.92.128.115/admin/product/fileUpload"
              目标服务器地址: 代理配置中 (vue.config.js)

            不允许跨域，就使用proxy
              action="/dev-api/admin/product/fileUpload"
              /dev-api -> request.js 代理
             在main.js中定义 Vue.prototype.$BASE_API = process.env.VUE_APP_BASE_API
           -->
          <el-upload
            class="avatar-uploader"
            :action="`${$BASE_API}/admin/product/fileUpload`"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img
              v-if="trademarkForm.logoUrl"
              :src="trademarkForm.logoUrl"
              class="avatar"
            />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <span>只能上传jpg/png图片，并且不能超过50kb</span>
        </el-form-item>
      </el-form>
      <span class="dialog-footer" slot="footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary" @click="submitForm('trademarkForm')"
          >确定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'TradeMarkList',
  data() {
    return {
      tradeAllList: [], //所有数据
      count: 0, //测试数据
      total: 0, //总数
      page: 1, //页码
      limit: 3, //每页条数
      visible: false, //对话框的显示隐藏
      trademarkForm: {
        tmName: '',
        logoUrl: '',
      },
      rules: {
        tmName: [
          {
            validator: this.validator,
            trigger: 'blur',
          },
        ],
        logoUrl: [{ required: true, message: '请上传品牌LOGO' }],
      },
    }
  },
  methods: {
    //删除当前品牌的信息
    del(row){
      this.$confirm(`您确认删除${row.tmName}`,"提示",{
        type:"warning",
      }).then(async()=>{
      await  this.$API.trademark.deleteTradmark(row.id)
      this.$message({
        type:'success',
        message:"删除成功"
      });
        this.getpageList(
          this.tradeAllList.length === 1 && page>1?
          this.page-1:this.page,
          this.limit
        );
      })
      .catch((error)=>{
        if(error === "cancel"){
          this.$message({
            type:"info",
            message:"已取消删除"
          })
        }
      })
    },
    //修改当前品牌信息
    update(row) {
      //清空表单验证
      this.$refs.trademarkForm && this.$refs.trademarkForm.clearValidate()
      this.visible = true
      this.trademarkForm = { ...row }
    },
    async getpageList(page, limit) {
      const result = await this.$API.trademark.getPageList(page, limit)
      if (result.code === 200) {
        this.$message.success('获取品牌分页列表成功')
        this.limit = result.data.size //获取每页显示的条数
        this.page = result.data.current //代表当前页码
        this.tradeAllList = result.data.records
        this.total = result.data.total //总数
      } else {
        this.$message.error('获取品牌分页列表失败')
      }
    },
    add() {
      this.visible = true
      //清空表单的校验
      this.$refs.trademarkForm && this.$refs.trademarkForm.clearValidate()
      //清空（从修改到添加  要先清空修改的数据）
      this.trademarkForm = {
        tmName: '',
        logoUrl: '',
      }
    },
    handleAvatarSuccess(res) {
      //上传图片成功的回调,这里的res.data代表的图片地址
      this.trademarkForm.logoUrl = res.data
    },
    beforeAvatarUpload(file) {
      const imgTypes = ['image/jpg', 'image/png', 'image/jpeg']
      //检测文件类型
      const isValidType = imgTypes.indexOf(file.type) > 1
      //检测文件大小 表示文件大小  不大于50kb
      const islt = file.size / 1024 < 50
      if (!isValidType) {
        this.$message.error('上传品牌LOGO 只能是 JPG 或者 PNG 格式')
      }
      if (!islt) {
        this.$message.error('上传品牌图片大小不能大于50kb')
      }
      //返回值为true，代表可以上传
      //返回值为false 代表不可上传
      return isValidType && islt
    },
    validator(rule, value, callback) {
      /*
        rule  校验的字段名
        value 校验的字段值
        callback 决定表单校验成功/失败
      */
      if (!value) {
        callback(new Error('请输入品牌名称'))
        return
      } else if (value.length < 2 || value.length > 10) {
        callback(new Error('输入品牌名称应为2-10位'))
        return
      }
      callback()
    },
    //提交表单
    submitForm(form) {
      //校验表单
      this.$refs[form].validate(async (valid) => {
        if (valid) {
          const { trademarkForm } = this
          const isUpdate = !!trademarkForm.id
          //代表是否更新
          if (isUpdate) {
            //如果是修改需要验证
            const tm = this.tradeAllList.find(
              (tm) => tm.id === trademarkForm.id
            )
            if (
              tm.tmName === trademarkForm.tmName &&
              tm.logoUrl === trademarkForm.logoUrl
            ) {
              this.$message.warning('不能提交与之前一样的表单')
              return
            }
          }
          let result
          if (isUpdate) {
            result = await this.$API.trademark.updateTrademark(trademarkForm)
          } else {
            result = await this.$API.trademark.addTrademark(trademarkForm)
          }
          if (result.code === 200) {
            this.$message.success(`${isUpdate ? '修改' : '添加'}品牌数据成功`)
            this.visible = false
            this.getpageList(this.page, this.limit)
          } else {
            this.$message.error(result.message)
          }
        }
      })
    },
  },
  mounted() {
    this.getpageList(1, 3)
  },
}
</script>
/*
lang="stylus"
可以省略{}
可以省略：
可以省略；
lang="sass"
如果写成scss 就必须改写的都写
可以省略{}
可以省略；
lang="less"
可以嵌套写
*/


<style lang="sass" scoped>
.trademark-img
  width: 150px
.trademark-pagination
  text-align: right
>>>.el-pagination__sizes
  margin-left: 250px

>>>  .avatar-uploader .el-upload
  border: 1px dashed #d9d9d9
  border-radius: 6px
  cursor: pointer
  position: relative
  overflow: hidden

>>> .avatar-uploader .el-upload:hover
  border-color: #409EFF

>>>  .avatar-uploader-icon
  font-size: 28px
  color: #8c939d
  width: 178px
  height: 178px
  line-height: 178px
  text-align: center

>>> .avatar
  width: 178px
  height: 178px
  display: block
</style>
