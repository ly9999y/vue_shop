<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 头部警告区域 -->
      <el-alert title="注意：只允许为三级分类设置参数！"
                type="warning"
                :closable="false"
                show-icon>
      </el-alert>
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader v-model="selectedCateKeys"
                       expand-trigger="hover"
                       :options="cateList"
                       :props="cateProps"
                       @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
      <!-- tab页 -->
      <el-tabs v-model="activeName"
               @tab-click="handleTabClick">
        <!-- 添加参数的的面板 -->
        <el-tab-pane label="动态参数"
                     name="many">
          <!-- 添加参数的按钮 -->
          <el-button type="primary"
                     :disabled="isBtnDisabled"
                     size="mini"
                     @click="addDialogVisible=true">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData"
                    border
                    stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环生成标签 -->
                <el-tag :key="index"
                        closable
                        @close="handleTagClose(index, scope.row)"
                        v-for="(item, index) in scope.row.attr_vals">{{item}}</el-tag>
                <!-- 可输入标签 -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary"
                           icon="el-icon-edit"
                           size="mini"
                           @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           icon="el-icon-delete"
                           size="mini"
                           @click="removeParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加属性的面板 -->
        <el-tab-pane label="静态属性"
                     name="only">
          <!-- 添加属性的按钮 -->
          <el-button type="primary"
                     :disabled="isBtnDisabled"
                     size="mini"
                     @click="addDialogVisible=true">添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData"
                    border
                    stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环生成标签 -->
                <el-tag :key="index"
                        closable
                        @close="handleTagClose(index, scope.row)"
                        v-for="(item, index) in scope.row.attr_vals">{{item}}</el-tag>
                <!-- 可输入标签 -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="saveTagInput"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称"
                             prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary"
                           icon="el-icon-edit"
                           size="mini"
                           @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           icon="el-icon-delete"
                           size="mini"
                           @click="removeParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog :title="'添加'
                           +
                           titleText"
               :visible.sync="addDialogVisible"
               width="50%"
               @close="addDialogClosed">
      <!-- 添加参数的表单 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数的对话框 -->
    <el-dialog :title="'修改' + titleText"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editDialogClosed">
      <!-- 添加参数的表单 -->
      <el-form :model="editForm"
               :rules="editFormRules"
               ref="editFormRef"
               label-width="100px">
        <el-form-item :label="titleText"
                      prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 级联选择框数据
      cateList: [],
      // 级联选择框属性
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedCateKeys: [],
      // 激活的tabs名称
      activeName: 'many',
      // 动态参数表格数据
      manyTableData: [],
      // 静态属性表格数据
      onlyTableData: [],
      // 添加参数的对话框显示
      addDialogVisible: false,
      // 添加参数的表单
      addForm: { attr_name: '' },
      // 添加参数表单校验规则
      addFormRules: {
        attr_name:
          [
            { required: true, message: '请输入名称', trigger: 'blur' }
          ]
      },
      // 修改参数对话框显示
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name:
          [
            { required: true, message: '请输入名称', trigger: 'blur' }
          ]
      },
      // 控制标签和输入框的切换显示
      inputVisible: false,
      // 文本框中输入的内容
      inputValue: ''
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data
    },
    async handleChange () {
      this.getParamsData()
    },
    handleTabClick () {
      this.getParamsData()
    },
    async getParamsData () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      res.data.forEach(element => {
        element.attr_vals = element.attr_vals
          ? element.attr_vals.split(' ') : []
        element.inputVisible = false
        element.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(`categories/
        ${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          })

        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success('添加成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    async showEditDialog (id) {
      const { data: res } = await this.$http.get(`categories/
        ${this.cateId}/attributes/${id}`,
        {
          params: { attr_sel: this.activeName }
        })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(`categories/
        ${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_sel: this.activeName,
            attr_name: this.editForm.attr_name
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success('修改成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    async removeParam (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该属性, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }

      const { data: res } = await this.$http.delete(`categories/
        ${this.cateId}/attributes/${id}`)

      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success('删除成功')
      this.getParamsData()
    },
    // 输入框失去焦点或按下回车键
    async handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrValues(row)
    },
    showInput (row) {
      row.inputVisible = true
      // $nextTick方法作用：当前页面元素被重新渲染之后，才会调用指定回调函数的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 将对attr_vals的操作保存到数据库
    async saveAttrValues (row) {
      const { data: res } = await this.$http.put(`categories/
        ${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      return this.$message.success('修改成功')
    },
    handleTagClose (index, row) {
      row.attr_vals.splice(index, 1)
      this.saveAttrValues(row)
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.input-new-tag {
  width: 100px;
}
</style>
