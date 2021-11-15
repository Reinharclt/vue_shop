<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row :class="['bdtop', i1 === 0 ? 'bdtop' : 'bdbottom', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过 for 循环嵌套渲染二级权限 -->
                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)">
                      {{ item2.authName }}
                      </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)">
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template v-slot="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRoleDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 添加角色的对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addRoleDialogVisible"
        width="50%"
        @close="addRoleDiaClosed">
        <!-- 内容主体区域 -->
        <el-form
          :model="addRoles"
          :rules="addRolesRules"
          ref="addRolesRef"
          label-width="90px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRoles.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addRoles.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addRolesDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 编辑权限的对话框 -->
      <el-dialog
        title="编辑权限"
        :visible.sync="editRolesDialogVisible"
        width="50%"
        @close="editRolesDiaClosed">
        <!-- 内容主体区域 -->
        <el-form
          :model="editRoles"
          :rules="editRolesRules"
          ref="editRolesRef"
          label-width="90px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRoles.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editRoles.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editRolesDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRoleInfo">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 分配权限的对话框 -->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%"
        @close="setRightDialogClosed">
        <!-- 内容主体区域 -->
        <!-- 树形控件 -->
        <el-tree
          show-checkbox
          :data="rightslist"
          :props="treeProps"
          node-key="id"
          default-expand-all
          :default-checked-keys="defKeys"
          ref="treeRef">
        </el-tree>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolelist: [],
      // 控制添加角色对话框的显示与隐藏
      addRoleDialogVisible: false,
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 添加角色的表单数据
      addRoles: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色表单的验证规则对象
      addRolesRules: {
        roleName: [
          { required: true, trigger: 'blur', message: '请输入角色名称!' }
        ],
        roleDesc: [
          { required: true, trigger: 'blur', message: '请输入角色描述!' }
        ]
      },
      // 控制编辑权限对话框的显示与隐藏
      editRolesDialogVisible: false,
      // 查询角色的表单数据
      editRoles: {},
      // 编辑角色表单的验证规则对象
      editRolesRules: {
        roleName: [
          { required: true, trigger: 'blur', message: '请输入角色名称!' }
        ],
        roleDesc: [
          { required: true, trigger: 'blur', message: '请输入角色描述!' }
        ]
      },
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: []
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    async getRoleList () {
      // 获取所有角色的列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败!')
      }
      this.rolelist = res.data
      // console.log(this.rolelist)
    },
    addRoleDiaClosed () {
      this.$refs.addRolesRef.resetFields()
    },
    addRole () {
      this.$refs.addRolesRef.validate(async valid => {
        if (!valid) return false
        // 可有发起添加角色的请求
        const { data: res } = await this.$http.post('roles', this.addRoles)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败!')
        }
        this.$message.success('添加角色成功!')
        this.getRoleList()
        // 隐藏添加角色的对话框
        this.addRoleDialogVisible = false
      })
    },
    // 展示角色编辑的对话框
    async showEditRoleDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色信息失败!')
      }
      this.editRoles = res.data
      this.editRolesDialogVisible = true
    },
    editRolesDiaClosed () {
      this.$refs.editRolesRef.resetFields()
    },
    // 修改用户信息并提交
    editRoleInfo () {
      this.$refs.editRolesRef.validate(async valid => {
        if (!valid) return false
        // 发送修改用户信息请求
        const { data: res } = await this.$http.put('roles/' + this.editRoles.roleId, { roleName: this.editRoles.roleName, roleDesc: this.editRoles.roleDesc })
        if (res.meta.status !== 200) {
          return this.$message.error('编辑并提交角色信息失败!')
        }
        this.editRolesDialogVisible = false
        this.getRoleList()
        this.$message.success('编辑并提交角色信息成功!')
      })
    },
    // 根据 Id 删除对应的用户信息
    async removeRoleById (id) {
      // 弹窗询问是否删除
      const confirmRoleResult = await this.$confirm('此操作将永久删除该用户信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消删除，则返回值为字符串 cancel
      if (confirmRoleResult !== 'confirm') {
        return this.$message.info('已取消删除!')
      }

      const { data: res } = await this.$http.delete('roles/' + id)

      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败!')
      }
      this.$message.success('删除用户成功!')
      this.getRoleList()
    },
    // 根据 Id 删除对应的权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否需要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除操作!')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除取消失败!')
      }

      // this.getRoleList()  不建议   会发生角色权限的完整渲染
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取所有权限数据失败!')
      }

      // 获取到的权限数据保存到 data 中
      this.rightslist = res.data
      console.log(this.rightslist)

      // 递归获取三级节点的 Id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的 id，并保存到 defKeys 数组中
    getLeafKeys (node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      console.log(keys)

      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配角色权限失败!')
      }
      this.$message.success('分配角色权限成功!')
      this.getRoleList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
