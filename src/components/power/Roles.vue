<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>    
      <!-- 角色列表 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1, i1) in scope.row.children" :key="item1.id" :class="['bdbottom', 'vcenter', i1 === 0?'bdtop':'']">
              <!-- 一级权限 -->
              <el-col :span="5" >
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="['vcenter',i2 === 0?'':'bdtop']">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" :class="[i3 === 0?'':'bdtop']" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <pre>
              {{scope.row}}
            </pre>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>        
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>        
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column> 
      </el-table>
    </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="35%"
      @close="addDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addForm" ref="addFormRef" label-width="80px" :rules="addFormRules">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色的对话框 -->
    <el-dialog
      title="修改角色信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改权限的对话框 -->
    <el-dialog
      title="修改角色权限"
      :visible.sync="showSetDialogVisible"
      width="30%"
      @close="setSetRightDialogClosed">
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer">
        <el-button @click="showSetDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色
      rolelist: [],
      // 控制添加角色对话框的显示和隐藏
      addDialogVisible: false,
      // 添加角色的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }, 
          { min: 2, max: 10, message: '角色名称长度在2~10之间', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }, 
          { min: 5, max: 10, message: '角色描述长度在5~10之间', trigger: 'blur' }
        ]
      },
      // 控制修改角色对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的角色信息对象
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }, 
          { min: 2, max: 10, message: '角色名称长度在2~10之间', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }, 
          { min: 5, max: 10, message: '角色描述长度在5~10之间', trigger: 'blur' }
        ]
      },
      // 控制修改角色权限对话框的显示与隐藏
      showSetDialogVisible: false,
      // 角色权限列表
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前即将分配权限角色id
      roleId: ''
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    async getRoleList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
    },
    // 监听添加角色对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮,添加角色
    addRole () {
      this.$refs.addFormRef.validate(async valid => {
        if (valid !== true) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功！')
        // 隐藏添加角色的对话框
        this.addDialogVisible = false
        // 重新获取角色列表数据
        this.getRoleList()
      })
    },
    // 展示编辑角色的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)  
      if (res.meta.status !== 200) return this.$message.error('用户信息查询失败')  
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改角色对话框的关闭事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮， 修改角色
    editRole () {
      this.$refs.editFormRef.validate(async valid => {
        if (valid !== true) return
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc })
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色失败！')
        }
        // 隐藏修改角色的对话框
        this.editDialogVisible = false
        // 重新获取角色列表数据
        this.getRoleList()
        return this.$message.success('修改角色成功！')
      })
    },
    // 点击按钮，删除角色
    async removeRoleById (id) {
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?', 
        '提示', 
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 如果用户取消了删除，返回字符串cancel
      // 如果用户确认删除，返回字符串 confirm
      console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消了删除')
      }  
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除角色成功！')
      // 重新获取用户列表数据
      this.getRoleList()
    },
    // 根据id删除角色权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除权限！')
      }
      const { data: res } = await this.$http.delete('roles/' + role.id + '/rights/' + rightId)
      if (res.meta.status !== 200) {
        this.$message.error('获取删除角色权限失败')
      }
      this.$message.success('删除角色权限成功！')
      // 重新获取角色的权限即可，不用重新获取角色列表
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.rightslist = res.data
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)
      this.showSetDialogVisible = true
    },
    // 获取已有权限的id并构建成数组（递归）
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听权限管理按钮关闭事件
    setSetRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post('roles/' + this.roleId + '/rights', { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('角色权限更新失败')
      this.$message.success('分配权限成功')
      this.getRoleList()
      this.showSetDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px
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
