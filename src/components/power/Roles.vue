<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>

      <el-col>
        <!-- 角色列表区 -->
        <el-table :data="rolelist" border stripe>
          <!-- 展开列 -->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row
                :class="['bdtop', i1 === 0 ? 'bdbottom' : '', 'vcenter']"
                v-for="(item1, i1) in scope.row.children"
                :key="item1.id"
              >
                <!-- 渲染一级权限 -->
                <el-col :span="5">
                  <el-tag
                    closable
                    @close="removeRightById(scope.row, item1.id)"
                    >{{ item1.authName }}</el-tag
                  >
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 渲染二和三级权限 -->
                <el-col :span="19">
                  <!-- 通过for循环嵌套渲染二级权限 -->
                  <el-row
                    :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                    v-for="(item2, i2) in item1.children"
                    :key="item2.id"
                    closable
                    @close="removeRightById(scope.row, item2.id)"
                  >
                    <el-col :span="6">
                      <el-tag type="success">{{ item2.authName }}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag
                        type="warning"
                        :class="[i3 === 0 ? '' : 'bdtop']"
                        v-for="(item3, i3) in item2.children"
                        :key="item3.id"
                        closable
                        @close="removeRightById(scope.row, item3.id)"
                        >{{ item3.authName }}</el-tag
                      >
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>

              <!-- <pre>{{ scope.row }}</pre> -->
            </template>
          </el-table-column>
          <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="角色名称" prop="roleName"></el-table-column>
          <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button
                size="mini"
                type="primary"
                icon="el-icon-edit"
                @click="showEditDialog(scope.row.id)"
                >编辑</el-button
              >
              <el-button
                size="mini"
                type="danger"
                icon="el-icon-delete"
                @click="removeRoleById(scope.row.id)"
                >删除</el-button
              >
              <el-button
                size="mini"
                type="warning"
                icon="el-icon-setting"
                @click="showSetRightDialog(scope.row)"
                >分配权限</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-card>

    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 node-key: 绑定选中的id -->
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加角色的对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体区 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="80px"
        @close="addDialogClosed"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色的对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="80px"
      >
        <el-form-item label="角色名称">
          <el-input v-model="editForm.roleName" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
      },
      // 所有列表数据
      rolelist: [],
      // 控制添加角色对话框的显示与隐藏
      addDialogVisible: false,
      // 添加角色的表单数据
      addForm: {
        roleName: '',
        roleDesc: '',
      },
      // 添加表单的验证规则对象
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }, // blur: 失去焦点
          {
            min: 3,
            max: 10,
            message: '角色名称的长度在(3, 10)',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          { required: true, message: '请描述角色', trigger: 'blur' },
          { min: 8, max: 30, message: '角色的长度在(8, 30)', trigger: 'blur' },
        ],
      },
      // 控制分配权限的对话框显示与隐藏
      setRightDialogVisible: false,
      // 控制修改角色对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的角色信息
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }, // blur: 失去焦点
          {
            min: 3,
            max: 10,
            message: '角色名称的长度在(3, 10)',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          { required: true, message: '请描述角色', trigger: 'blur' },
          { min: 8, max: 30, message: '角色的长度在(8, 30)', trigger: 'blur' },
        ],
      },
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children',
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前即将分配权限的角色Id
      roleId: '',
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败!')
      }

      this.rolelist = res.data
      console.log(this.rolelist)
    },
    // 监听添加角色对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮, 添加新角色
    addRole() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return

        // 可以发起添加角色的网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色失败!')
        }

        this.$message.success('添加角色成功!')
        // 隐藏添加角色的对话框
        this.addDialogVisible = false
        // 重新会哦去角色列表数据
        this.getRolesList()
      })
    },
    // 展示编辑角色的对话框
    async showEditDialog(id) {
      // console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色信息失败!')
      }
      this.editForm = res.data

      this.editDialogVisible = true
    },
    // 监听修改角色对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改角色信息并提交
    editRoleInfo() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        // 发起修改用户信息的数据请求
        const { data: res } = await this.$http.put(
          'roles/' + this.editForm.id,
          {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc,
          }
        )
        if (res.meta.status !== 400) {
          return this.$message.error('更新角色信息失败!')
        }
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('更新角色信息成功!')
      })
    },
    // 根据ID删除对应的角色信息
    async removeRoleById(id) {
      // 询问用户是否删除信息
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)

      // 若用户确认删除, 返回字符串 confirm
      // 若用户取消删除, 放回字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败!')
      }

      this.$message.success('删除角色成功!')
      this.getRolesList()
    },
    // 根据Id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹窗提示角色是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除!')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )

      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败!')
      }

      // this.getRolesList()
      role.children = res.data
    },
    // 展示分配权限的显示框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败!')
      }

      // 把获取到权限数据保存到 data
      this.rightslist = res.data
      console.log(this.rightslist)

      // 递归获取三级节点
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 递归, 获取角色下所有三级权限的id, 保存到defKeys数组中
    getLeafKeys(node, arr) {
      // 当前node节点不包含children属性, 则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach((item) => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(), // ...放在此数组中

        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ]

      console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败!')
      }
      this.$message.success('分配权限成功!')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
  },
}
</script>

<style lang="less" scoped>
.bdtop {
  border-top: 1px solid #eee;
}

.bdbuttom {
  border-bottom: 1px solid #eee;
}

.el-tag {
  margin: 7px;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
