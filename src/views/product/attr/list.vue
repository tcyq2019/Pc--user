<template>
  <div>
    <Category @change="getAttrList" :disabled="!isShowList" />
    <el-card v-show="isShowList" style="margin-top: 20px">
      <el-button type="primary" icon="el-icon-plus">添加属性</el-button>

      <el-table :data="attrList" border style="width: 100%; margin: 20px 0">
        <el-table-column type="index" label="序号" width="80" align="center">
        </el-table-column>
        <el-table-column prop="attrName" label="属性名称" width="150">
        </el-table-column>

        <el-table-column label="属性值列表">
          <template v-slot="{ row }">
            <el-tag
              style="margin-right: 5px"
              v-for="attrVal in row.attrValueList"
              :key="attrVal.id"
              >{{ attrVal.valueName }}</el-tag
            >
          </template>
        </el-table-column>
        <el-table-column label="操作" width="150">
          <template v-slot="{ row }">
            <el-button
              type="warning"
              icon="el-icon-edit"
              size="mini"
              @click="update(row)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-card v-show="!isShowList" style="margin-top: 20px">
      <el-form :model="attr" inline>
        <el-form-item label="属性名" prop="attrName">
          <el-input v-model="attr.attrName"></el-input>
        </el-form-item>
      </el-form>
      <el-button type="primary" icon="el-icon-plus" @click="addValueList"
        >添加属性</el-button
      >

      <el-table
        :data="attr.attrValueList"
        border
        style="width: 100%; margin: 20px 0"
      >
        <el-table-column type="index" label="序号--" width="80" align="center">
        </el-table-column>
        <!--
          事件修饰符 ：
          .native
          专门给组件绑定事件用的
          会给组件中的第一个标签
          绑定相应的原生Dom事件
          会穿透组件给她绑定原生dom事件
         -->
        <el-table-column label="属性值名称">
          <template v-slot="{ row, $index }">
            <el-input
              v-if="row.edit"
              v-model="row.valueName"
              @blur="editCoponent(row, $index)"
              @keyup.enter.native="row.edit = false"
              autofocus
              ref="input"
              size="mini"
            ></el-input>
            <!-- 直接给对象添加数据不会是响应式的因为数据改变和画面不会变，所有需要勇$set去设置才会是响应式的 -->
            <span
              v-else
              @click="edit(row)"
              style="display: block; width: 100%"
              >{{ row.valueName }}</span
            >
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot="{ row, $index }">
            <el-popconfirm
              :title="`确定删除${row.valueName}？`"
              @onConfirm="delAttrValue($index)"
            >
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                slot="reference"
              ></el-button>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>
      <el-button type="primary" @click="save">保存</el-button>
      <el-button @click="isShowList = true">取消</el-button>
    </el-card>
  </div>
</template>

<script>
import Category from './category'
export default {
  name: 'AttrList',
  data() {
    return {
      attrList: [],
      isShowList: true,
      attr: {
        attrName: '',
        attrValueList: [],
      },
    }
  },
  methods: {
    //处理添加属性为空的bug
    editCoponent(row, index) {
      if (!row.valueName) {
        this.attr.attrValueList.splice(index, 1)
        return
      }
      row.edit = false
    },
    //保存所有属性
    async save() {
      const result = await this.$API.attrs.saveAttrInfo(this.attr)
      if (result.code === 200) {
        this.$message.success('跟新属性成功')
        this.isShowList = true
        this.getAttrList(this.category)
      } else {
        this.$message.error(result.message)
      }
    },
    //删除属性
    delAttrValue(index) {
      this.attr.attrValueList.splice(index, 1)
    },
    //添加属性
    addValueList() {
      this.attr.attrValueList.push({
        edit: true,
      })
      this.$nextTick(() => {
        this.$refs.input.focus()
      })
    },
    //span和input切换
    edit(row) {
      this.$set(row, 'edit', true)
      this.$nextTick(() => {
        this.$refs.input.focus()
      })
    },
    update(attr) {
      //浅克隆会在你改数据时修改原数据，这里需要用到深克隆
      /*   this.attr = {
        ...attr,
      } */
      this.attr = JSON.parse(JSON.stringify(attr))
      this.isShowList = false
    },

    async getAttrList(category) {
      this.category = category
      const result = await this.$API.attrs.getAttrList(category)
      console.log(result)
      if (result.code === 200) {
        this.attrList = result.data
      } else {
        this.$message.error(result.message)
      }
    },
  },
  components: {
    Category,
  },
}
</script>
