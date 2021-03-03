<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert :closable="false" title="注意:只允许第三级分类设置相关参数" show-icon type="warning"></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cateProps"
            @change="cateHandleChange">
            </el-cascader>
        </el-col>
      </el-row>
      <!-- 页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisable" @click="addDialogVisible = true">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable>{{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"></el-input>
                <!-- 添加按钮 -->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisable" @click="addDialogVisible = true">添加属性</el-button>
          <!-- 静态参数表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加动态/静态对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 添加参数的对话框 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name" :rule="addFormRules">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <!-- 添加参数的对话框 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name" :rule="editFormRules">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有商品列表
      catelist: [],
      // 级联选择框的配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 级联选择框双向绑定的数组
      selectedCateKeys: [],
      // 被激活的页签的名称
      activeName: 'many',
      // 动态参数的数据
      manyTableData: [],
      // 静态参数的数据
      onlyTableData: [],
      // 控制添加对话框的显示和隐藏
      addDialogVisible: false,
      // 添加参数的表单数据对象
      addForm: {
        attr_name: ''
      },
      // 添加表单杜的验证规则对象
      addFormRules: {
        attr_name: [
          {
            required: true, message: '请输入参数名称', trigger: 'blur'
          }
        ]
      },
      // 修改对话框的显示与隐藏
      editDialogVisible: false,
      // 修改对话框的数据对象
      editForm: {
      },
      // 修改参数的验证规则对象
      editFormRules: {
        attr_name: [
          {
            required: true, message: '请输入参数名称', trigger: 'blur'
          }
        ]
      }
    }
  },
  created () {
    // 获取所有商品分类列表
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      this.catelist = res.data
    },
    // 级联选择框选中项发生变化触发函数
    async cateHandleChange () {
      // 证明选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
      } else {
        this.getParamsDate()
      }
    },
    // tab 页签点击事件的处理函数
    handleTabClick () {
      this.getParamsDate()
    },
    // 获取商品的参数
    async getParamsDate () {
      // 根据所选分类的id，和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes', { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取参数列表失败')
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 文本框中输入的值
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 监听添加对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮，添加属性，并关闭对话框
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories/' + this.cateId + '/attributes', {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.addDialogVisible = false
        this.getParamsDate()
      })
    },
    // 点击按钮，展示修改的对话框
    async showEditDialog (attrid) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(
        'categories/' + this.cateId + '/attributes/' + attrid, {
          params: { attr_sel: this.activeName }
        })
      if (res.meta.status !== 200) return this.$message.error('查询失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 重置修改对话框的显示与隐藏
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮，修改用户信息
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'categories/' + this.cateId + '/attributes/' + this.editForm.attr_id, {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName 
          }
        )
        if (res.meta.status !== 200) return this.$message.error('修改失败')
        this.$message.success('修改参数成功')
        this.getParamsDate()
        this.editDialogVisible = false
      })
    },
    // 删除参数项
    async removeParams (attrid) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 用户取消了删除的操作
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('categories/' + this.cateId + '/attributes/' + attrid)
      if (res.meta.status !== 200) return this.$message.error('删除参数失败')
      this.$message.success('删除参数成功')
      this.getParamsDate()
    },
    // 文本框失去焦点，或者按下Enter键，都会触发
    async handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return false
      }
      // 如果没有return 证明输入的内容，需要做后续处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 发起保存，提交参数
      const { data: res } = await this.$http.put('categories/' + this.cateId + '/attributes/' + row.attr_id, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) return this.$message.error('修改失败')
      this.$message.success('修改成功')
    },
    // 点击按钮，展示文本输入框
    showInput (row) {
      row.inputVisible = true
      // 文本框自动获取焦点
      // nextTick 作用：当页面上元素被重新渲染后，才执行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true,否则返回false
    isBtnDisable () {
      if (this.selectedCateKeys.length !== 3) return true
      return false
    },
    // 当前选中的三级分类id
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 动态计算标题
    titleText () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0
}
.el-tag {
  margin: 5px 10px 5px 0
}
.input-new-tag {
  width: 120px
}
</style>
