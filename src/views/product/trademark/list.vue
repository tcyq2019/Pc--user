<template>
  <div>
    <el-button type="primary" icon="el-icon-plus" @click="add">添加</el-button>

    <el-table :data="trademarkList" border style="width: 100%; margin: 20px 0">
      <el-table-column type="index" label="序号" width="80" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称"> </el-table-column>
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
          <el-button
            type="warning"
            icon="el-icon-edit"
            @click="putTradeMark(row)"
            >修改</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            @click="delTradeMark(row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="trademark-pagination"
      @size-change="getPageList(page, $event)"
      @current-change="getPageList($event, limit)"
      :page-sizes="[3, 6, 9]"
      :page-size.sync="limit"
      :current-page="page"
      layout="prev, pager, next, jumper, sizes, total"
      :total="total"
    >
    </el-pagination>

    <el-dialog
      :title="`${trademarkForm.id ? '修改' : '添加'}品牌`"
      :visible.sync="Visible"
      width="50%"
    >
      <el-form
        :model="trademarkForm"
        ref="trademarkForm"
        :rules="rules"
        label-width="100px"
      >
        <el-form-item label="品牌名称" prop="logoUrl">
          <el-input v-model="trademarkForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌LOGO" prop="logoUrl">
          <!--
            前提允许跨域
            action="http://182.92.128.115/admin/product/fileUpload"
            目标服务器地址：代理配置中（vue.config.js）

            不允许跨域，就使用proxy
            action="/dev-api/admin/product/fileUpload"
            /dev-api->request.js代理
            再mian.js中定义 Vue.prototype.$BASE_API = process.env.VUE_APP_BASE_API


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
        </el-form-item>
        <span>请上传jpg/png格式的图片,并且不能超过50kb,我要求就是这么多</span>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="Visible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm('trademarkForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { valid } from 'mockjs'
export default {
  name: 'TrademarkList',
  data() {
    return {
      trademarkList: [],
      total: 0,
      page: 1,
      limit: 3,
      Visible: false,
      trademarkForm: {
        tmName: '',
        logoUrl: '',
      },
      rules: {
        tmName: [
          {
            /* required: true, message: '请输入品牌名称' */
            validator: this.validator,
            trigger: 'blur',
          },
        ],
        logoUrl: [{ required: true, message: '请上传品牌LOGO' }],
      },
    }
  },
  methods: {
    validator(rule, value, callback) {
      /*
      rule校验的名字
      value 校验的字段直
      callback 决定表单校验成功/失败
      */
      if (!value) {
        callback(new Error('请输入品牌名称'))
        return
      } else if (value.length < 2 || value.length > 10) {
        callback(new Error('输入品牌名称的长度为2-10位'))
        return
      }
      /* callback里面如果是写的error就是错误回调 如果是callback()就是成功回调 */
      callback()
    },
    add() {
      this.$refs.trademarkForm && this.$refs.trademarkForm.clearValidate()
      this.Visible = true
      this.trademarkForm = {
        tmName: '',
        logoUrl: '',
      }
    },
    putTradeMark(row) {
      this.Visible = true
      this.trademarkForm = { ...row }
    },
    //删除
    /* 第一种方法 async delTradeMark(id) {
      if (window.confirm('您确定删除该品牌吗?')) {
        await this.$API.trademark.deleteTradmark(id)
        console.log(id)
      }
      this.getPageList(this.page, this.limit)
      this.$message({
        type:"success",
        message:"删除成功"
      })
    }, */
    /* 老师方法 */
    delTradeMark(row) {
      this.$confirm(`确定删除${row.tmName}吗？`, '提示', {
        type: 'warning',
      })
        .then(async () => {
          //点击确定的回调
          //发送删除品牌的请求
          const result = await this.$API.trademark.deleteTradmark(row.id)
          //如果成功了，提示成功，重新获取列表（那一页)
          this.$message({
            type: 'success',
            message: '删除成功',
          })
          //那一页？显示上一页（删除时当前列表数据只剩一页）否则显示当前列表数据
          //如果当前是第一页并且只剩下一条数据==>去请求的第一页的数据（也就是当前页）
          this.getPageList(
            this.trademarkList.length === 1 && this.page > 1
              ? this.page - 1
              : this.page,
            this.limit
          )
        })
        .catch((error) => {
          if (error === 'cancel') {
            this.$message({
              type: 'info',
              message: '已经取消删除',
            })
          }
        })
    },

    submitForm(form) {
      //校验表单
      this.$refs[form].validate(async (valid) => {
        if (valid) {
          const { trademarkForm } = this
          const isUpdata = !!trademarkForm.id
          if (isUpdata) {
            const tm = this.trademarkList.find(
              (tm) => tm.id === trademarkForm.id
            )
            if (
              tm.tmName === trademarkForm.tmName &&
              tm.logoUrl === trademarkForm.logoUrl
            ) {
              this.$message.warning('不能提交与之前一样的额数据')
              return
            }
          }
          //表单验证
          //发送请求
          let result
          if (isUpdata) {
            result = await this.$API.trademark.updateTrademark(trademarkForm)
          } else {
            result = await this.$API.trademark.addTrademark(trademarkForm)
          }

          if (result.code === 200) {
            this.$message.success('添加品牌数据成功')
            this.Visible = false
            this.getPageList(this.page, this.limit)
          } else {
            this.$message.error(result.message)
          }
        }
      })
    },
    //上传图片成功的回调

    /*  handleAvatarSuccess(res) {
      this.trademarkForm.logoUrl = res.data
      console.log(res)
    }, */
    handleAvatarSuccess(res) {
      this.trademarkForm.logoUrl = res.data
    },
    //上传图片之前的回调
    beforeAvatarUpload(file) {
      const imgType = ['image/jpg', 'image/png', 'image/jpeg']
      //检测文件类型
      const isValidType = imgType.indexOf(file.type) > -1
      //检测文件大小
      const isLt = file.size / 1024 < 50
      if (!isValidType) {
        this.$message.error('上传图片只能是 JPG 或者 PNG 格式！')
      }
      if (!isLt) {
        this.$message.error('上传图片不能超过50kb')
      }
      return isValidType && isLt
    },
    //请求分页列表数据
    async getPageList(page, limit) {
      try {
        const result = await this.$API.trademark.getPageList(page, limit)
        console.log(result)
        if (result.code === 200) {
          this.$message.success('请求成功')
          this.trademarkList = result.data.records
          this.total = result.data.total
          this.page = result.data.current
          this.limit = result.data.size
        } else {
          this.$message.error('获取品牌分页列表失败')
        }
      } catch (e) {
        this.$message.error('获取品牌分页列表失败')
      }
    },
  },
  mounted() {
    this.getPageList(this.page, this.limit)
  },
}
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
</script>

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
