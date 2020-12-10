<template>
  <div>
    <!--    导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCatDialog">添加分类</el-button>
        <!-- 树形表格 -->
          <tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false"
          :expand-type="false" show-index index-text="#" border>
            <template slot="isok" slot-scope="scope">
              <i class="el-icon-success" v-if="scope.row.cat_deleted === false"
              style="color: lightgreen"></i>
              <i class="el-icon-error" v-else style="color: red"></i>
            </template>
            <template slot="order" slot-scope="scope">
              <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
              <el-tag size="mini" type="success"
              v-else-if="scope.row.cat_level === 1">二级</el-tag>
              <el-tag size="mini" type="warning"
              v-else-if="scope.row.cat_level === 2">三级</el-tag>
            </template>
            <template slot="opt">
              <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
              <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            </template>
          </tree-table>
        <!-- 分页区域-->
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[2,3,5,10]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
          </el-pagination>
        </el-col>
      </el-row>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%" @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类" >
          <el-cascader
            v-model="selectKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            :clearable="true">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表
      catelist: [],
      // 总数据条数
      total: 0,
      // table 指定列的定义
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        // 表示自定义模板
        type: 'template',
        // 表示当前模板使用名称
        template: 'isok'
      }, {
        label: '排序',
        // 表示自定义模板
        type: 'template',
        // 表示当前模板使用名称
        template: 'order'
      }, {
        label: '操作',
        // 表示自定义模板
        type: 'template',
        // 表示当前模板使用名称
        template: 'opt'
      }],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        // 父级id
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      cascaderProps: {
        checkStrictly: true,
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类id数组
      selectKeys: []
    }
  },
  created() {
    this.getCatelist()
  },
  methods: {
    async getCatelist () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        this.$message.error(res.meta.msg)
      }
      this.catelist = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getCatelist()
    },
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getCatelist()
    },
    showAddCatDialog () {
      // 先获取父级分类的数据列表
      this.getParentCatelist()
      this.addCateDialogVisible = true
    },
    async getParentCatelist () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.parentCateList = res.data
      // console.log(this.parentCateList)
    },
    parentCateChanged () {
      // 如果selectKeys 数组中的length大于0，证明选中了父级分类
      // 反之就说明没有选中任何父级分类
      if (this.selectKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        this.addCateForm.cat_level = this.selectKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 添加新的分类
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return false
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success(res.meta.msg)
        this.getCatelist()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  }

}
</script>

<style lang="less" scoped>
.treeTable{
  margin-top: 20px;
}

.el-cascader{
  width: 100%;
}
</style>
