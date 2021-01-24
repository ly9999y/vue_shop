<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="showAddRoleDialog">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="rolesList"
                stripe="">
        <el-table-column label="*"
                         type="expand">
          <template slot-scope="scope">
            <el-row :class="['bd-buttom', index1===0?'bd-top':'', 'vcenter']"
                    v-for="(item1, index1) in scope.row.children"
                    :key="item1.id ">
              <el-col :span="5">
                <el-tag closable
                        @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row :class="['bd-buttom', index2===0?'bd-top':'', 'vcenter']"
                        v-for="(item2, index2) in item1.children"
                        :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success"
                            closable
                            @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag v-for="(item3) in item2.children"
                            type="warning"
                            :key="item3.id"
                            closable
                            @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"
                         label="#"></el-table-column>
        <el-table-column label="角色名称"
                         prop="roleName"></el-table-column>
        <el-table-column label="角色描述"
                         prop="roleDesc"></el-table-column>
        <el-table-column label="操作"
                         width="300px">
          <template slot-scope="scope">
            <el-button type="primary"
                       size="mini"
                       icon="el-icon-edit"
                       @click="showEditDialog(scope.row.id)">修改</el-button>
            <el-button type="danger"
                       size="mini"
                       icon="el-icon-delete"
                       @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button type="warning"
                       size="mini"
                       icon="el-icon-setting"
                       @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色"
               :visible.sync="addRoleDialogVisible"
               width="50%"
               @close="addRoleDialogClose">
      <el-form :model="addRoleForm"
               :rules="addRoleFormRules"
               ref="addRoleFormRef"
               label-width="100px">
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input type="textarea"
                    v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色对话框 -->
    <el-dialog title="修改角色"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editDialogClose">
      <el-form :model="editRoleForm"
               :rules="editRoleFormRules"
               ref="editRoleFormRef"
               label-width="100px">
        <el-form-item label="角色Id"
                      prop="roleId">
          <el-input v-model="editRoleForm.roleId"
                    disabled></el-input>
        </el-form-item>
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input type="textarea"
                    v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog title="提示"
               :visible.sync="setRightDialogVisible"
               width="50%"
               @close="setRightDialogClose">
      <!-- 树形控件 -->
      <el-tree :data="rightsList"
               :props="treeProps"
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="defaultKeys"
               ref="treeRef"></el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      // 角色列表数据
      rolesList: [],
      // 添加角色对话框显示标识
      addRoleDialogVisible: false,
      // 添加角色对话框表单数据
      addRoleForm: {},
      // 添加角色对话框校验规则
      addRoleFormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' },
        { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }],
        roleDesc: [
          { required: false, message: '请输入角色描述', trigger: 'blur' },
          { min: 0, max: 2000, message: '最长 2000 个字符', trigger: 'blur' }
        ]
      },
      // 编辑角色对话框显示标识
      editDialogVisible: false,
      // 编辑角色对话框数据
      editRoleForm: {},
      editRoleFormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' },
        { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }],
        roleDesc: [
          { required: false, message: '请输入角色描述', trigger: 'blur' },
          { min: 0, max: 2000, message: '最长 2000 个字符', trigger: 'blur' }
        ]
      },
      setRightDialogVisible: false,
      // 角色权限列表
      rightsList: [],
      // 树形控件属性设置
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 树形控件默认选中的节点
      defaultKeys: [],
      // 分配权限的角色ID
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色
    getRolesList: async function () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rolesList = res.data
    },
    // 展示添加角色对话框
    showAddRoleDialog () {
      this.addRoleDialogVisible = true
    },
    // 清除添加角色对话框表单数据
    addRoleDialogClose () {
      this.$refs.addRoleFormRef.resetFields()
    },
    // 添加角色
    async addRole () {
      const { data: res } = await this.$http.post('roles', { roleName: this.addRoleForm.roleName, roleDesc: this.addRoleForm.roleDesc })
      if (res.meta.status !== 201) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success('添加角色成功')
      // 刷新角色列表
      this.getRolesList()
      // 关闭添加角色对话框
      this.addRoleDialogVisible = false
      // 清空添加对话框表单
      this.$refs.addRoleFormRef.resetFields()
    },
    // 修改角色
    async showEditDialog (id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editRoleForm = res.data
    },
    // 清空修改角色表单
    editDialogClose () {
      this.$refs.editRoleFormRef.resetFields()
    },
    editRole () {
      this.$refs.editRoleFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editRoleForm.roleId, { roleName: this.editRoleForm.roleName, roleDesc: this.editRoleForm.roleDesc })
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        // 刷新角色列表
        this.getRolesList()
        // 关闭编辑角色对话框
        this.editDialogVisible = false
        // 清空编辑对话框表单
        this.$refs.editRoleFormRef.resetFields()
      })
    },
    async removeRoleById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success('角色删除成功')
      this.getRolesList()
    },
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)

      if (confirmResult !== 'confirm') {
        this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success('角色权限删除成功')
      role.children = res.data
    },
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightsList = res.data
      this.getLeafKeys(role, this.defaultKeys)
      this.setRightDialogVisible = true
    },
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(element => {
        this.getLeafKeys(element, arr)
      })
    },
    setRightDialogClose () {
      this.defaultKeys = []
    },
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idstr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idstr })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }

      this.$message.success('分配权限成功')
      this.setRightDialogVisible = false
      this.getRolesList()
    }
  }
}
</script>

<style lang="less" scoped>
.bd-top {
  border-top: 1px solid #eeeeee;
}
.bd-buttom {
  border-bottom: 1px solid #eeeeee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
