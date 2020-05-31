<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRole">添加角色</el-button>
        </el-col>
      </el-row>

      <el-table :data="roleList" style="width: 100%" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id" :class="['bdbottom',i1===0?'bdtop':'']">
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2===0?'':'bdtop']">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="13">
                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable
                      @close="removeRightById(scope.row,item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column prop="roleName" label="角色名称">
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述">
        </el-table-column>
        <el-table-column label="操作" width="350px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" @click="editRole(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" @click="showRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="30%" @close="setRightDialogClose">
      <el-tree :data="rightList" :props="treeProps" show-checkbox node-key="id" default-expand-all
        :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="40%" @close="addRoleDialogClose">
      <el-form :model="roleForm" :rules="roleFormRules" ref="roleFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="roleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="编辑角色" :visible.sync="editRoleDialogVisible" width="40%" @close="editRoleDialogClose">
      <el-form :model="editRoleForm" :rules="editRoleFormRules" ref="editRoleFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editSaveRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        roleList: [],
        setRightDialogVisible: false,
        editRoleDialogVisible: false,
        rightList: [],
        treeProps: {
          children: 'children',
          label: 'authName'
        },
        defKeys: [],
        roleId: '',
        addRoleDialogVisible: false,
        roleForm: {
          roleName: '',
          roleDesc: ''
        },
        roleFormRules: {
          roleName: [{
            required: true,
            message: '请输入角色名称',
            trigger: 'blur'
          }],
          roleDesc: [{
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          }]
        },
        editRoleFormRules: {
          roleName: [{
            required: true,
            message: '请输入角色名称',
            trigger: 'blur'
          }],
          roleDesc: [{
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          }]
        },
        editRoleForm: {
          roleName: '',
          roleDesc: ''
        }
      }
    },
    created() {
      this.getRoleList()
    },
    methods: {
      async getRoleList() {
        const {
          data: res
        } = await this.$http.get('roles')
        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败')
        }
        this.roleList = res.data
      },
      async removeRightById(role, rightId) {
        const confirmResult = await this.$confirm(
          '此操作将永久删除该角色, 是否继续?',
          '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        ).catch(err => err)
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        } else {
          const {
            data: res
          } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
          if (res.meta.status !== 200) {
            return this.$message.error('删除角色指定权限失败')
          } else {
            role.children = res.data
          }
        }
      },
      async showRightDialog(role) {
        const {
          data: res
        } = await this.$http.get('rights/tree')
        if (res.meta.status !== 200) {
          return this.$message.error('获取权限列表失败')
        }
        this.rightList = res.data
        this.getLeafKeys(role, this.defKeys)
        this.roleId = role.id
        this.setRightDialogVisible = true
      },
      getLeafKeys(node, arr) {
        if (!node.children) {
          return arr.push(node.id)
        }
        node.children.forEach(item => {
          this.getLeafKeys(item, arr)
        })
      },
      setRightDialogClose() {
        this.defKeys = []
      },
      async allotRights() {
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]
        const idStr = keys.join(',')
        const {
          data: res
        } = await this.$http.post(`roles/${this.roleId}/rights`, {
          rids: idStr
        })
        if (res.meta.status !== 200) {
          return this.$message.error('分配权限失败')
        }
        this.$message.success('分配权限成功')
        this.getRoleList()
        this.setRightDialogVisible = false
      },
      addRole() {
        this.addRoleDialogVisible = true
      },
      saveRole() {
        this.$refs.roleFormRef.validate(async valid => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.post('roles', this.roleForm)
          if (res.meta.status !== 201) {
            return this.$message.error('添加角色失败')
          }
          this.$message.success('添加角色成功')
          this.getRoleList()
          this.addRoleDialogVisible = false
        })
      },
      addRoleDialogClose() {
        this.$refs.roleFormRef.resetFields()
      },
      async deleteRole(roleId) {
        const confirmResult = await this.$confirm(
          '此操作将永久删除该角色, 是否继续?',
          '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        ).catch(err => err)
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        } else {
          const {
            data: res
          } = await this.$http.delete(`roles/${roleId}`)
          if (res.meta.status !== 200) {
            return this.$message.error('删除角色失败')
          }
          this.$message.success('删除角色成功')
          this.getRoleList()
        }
      },
      async editRole(roleId) {
        const {
          data: res
        } = await this.$http.get('roles/' + roleId)
        if (res.meta.status !== 200) {
          debugger
          return this.$message.error('获取角色失败')
        }
        this.roleId = roleId
        this.editRoleForm = res.data
        this.editRoleDialogVisible = true
      },
      editSaveRole() {
        this.$refs.editRoleFormRef.validate(async valid => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.put('roles/' + this.roleId, this.editRoleForm)
          if (res.meta.status !== 200) {
            return this.$message.error('编辑角色失败')
          }
          this.$message.success('编辑角色成功')
          this.getRoleList()
          this.editRoleDialogVisible = false
        })
      },
      editRoleDialogClose() {
        this.$refs.editRoleFormRef.resetFields()
      }
    }
  }

</script>

<style lang="less" scoped>
  .el-row {
    margin-bottom: 15px
  }

  .el-tag {
    margin: 7px;
  }

  .bdtop {
    border-top: 1px solid #eee
  }

  .bdbottom {
    border-bottom: 1px solid #eee
  }

  .el-row {
    display: flex;
    align-items: center
  }

  .el-form-item__label {
    text-align: left;
  }

</style>
