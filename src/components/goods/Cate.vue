<template>
  <div>
    <!--    面包屑导航区域-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品父类</el-breadcrumb-item>
    </el-breadcrumb>

    <!--    卡片试图区域-->
    <el-card >
      <!--      添加分类按钮区域-->
      <el-row>
        <el-col :pull="11" class="top-button">
          <el-button type="primary" @click="showAddCateDialog" >添加分类</el-button>
        </el-col>
      </el-row>
      <!--        商品分类列表区域-->
      <tree-table :data="cateList" :columns="columns" :selection-type="false"
        :expand-type="false" show-index index-text="#" border stripe class="treeTable">
<!--        是否有效-->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="!scope.row.cat_delete" style="color: green"></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
<!--        排序-->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
<!--        操作-->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"
                     @click="showEditDialog(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini"
                     @click="removeByCateId(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>
      <!--      分页区域-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

<!--    添加分类的对话框-->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"  @close="addCateDialogClosed">
      <el-form :model="addCateForm" :rules="addCateFormRules"
        ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged" size="mini" ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="addCate">确 定</el-button>
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
      </span>
    </el-dialog>
<!--    编辑分类的对话框-->
    <el-dialog title="修改分类" :visible="edidCateDialogVisible" width="50%"
      @close="editCateDialogClosed">
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef"
        label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="editCate">确 定</el-button>
        <el-button @click="edidCateDialogVisible = false">取 消</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  created () {
    this.getCateList()
  },
  data () {
    return {
      // 商品分类列表
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 10
      },
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 使用自定义模板列
          type: 'template',
          // 模板名称
          template: 'isok'
        },
        {
          label: '排序',
          // 使用自定义模板列
          type: 'template',
          // 模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 使用自定义模板列
          type: 'template',
          // 模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据
      addCateForm: {
        // 要添加的分类名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 当前分类的等级 默认是一级分类
        cat_level: 0
      },
      // 添加分类表单的规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类Id
      selectedKeys: [],
      // 控制修改分类的对话框的显示与隐藏
      edidCateDialogVisible: false,
      // 编辑分类的表单数据
      editCateForm: {
        // 分类id
        cat_id: '',
        // 要添加的分类名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 当前分类的等级 默认是一级分类
        cat_level: 0
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败！')
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize变化
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum变化
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 添加关闭添加对话框事件
    setRightDialogClosed () {
      this.addCateDialogVisible = false
    },
    // 点击按钮 显示添加分类的对话框
    showAddCateDialog () {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类数据失败！')
      this.parentCateList = res.data
    },
    // 选择性发生变化触发
    parentCateChanged () {
      // 如果selectedKeys 的length大于0 说明选中分类了
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 当前分类的等级
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮 添加新的分类
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error('添加分类失败！')
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置数据表单
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 展示编辑分类的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取分类信息失败！')
      console.log(res.data)
      this.editCateForm = res.data
      this.edidCateDialogVisible = true
    },
    // 关闭编辑用户的对话框
    editCateDialogClosed () {
      this.editCateForm.cat_name = ''
      this.edidCateDialogVisible = false
    },
    // 编辑确定 修改分类
    async editCate () {
      const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id,
        { cat_name: this.editCateForm.cat_name })
      if (res.meta.status !== 200) return this.$message.error('更新分类信息失败！')
      this.$message.error('更新分类信息成功！')
      this.getCateList()
      this.editCateDialogClosed()
    },
    // 删除分类
    async removeByCateId (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
        // 取消需要先捕获 再赋值给confirmResult
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败')
      }
      this.$message.success('删除分类成功')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }
  .top-button {
    margin-left: 9px;
  }
  .el-cascader {
    width: 100%;
  }
</style>
