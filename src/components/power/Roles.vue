<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
<!--    卡片视图-->
    <el-card>
      <!-- 添加角色-->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolelist" stripe border>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom',index1 === 0 ? 'bdtop': '', 'vcenter']" :key="item1.id" v-for="(item1,index1) in scope.row.children">
              <!-- 一级权限-->
              <el-col :span="5">
                <el-tag closable @click="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级权限和三级-->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限-->
                <el-row  :class="[index2 === 0 ? '': 'bdtop', 'vcenter']" :key="item2.id" v-for="(item2,index2) in item1.children">
                  <el-col :span="6">
                    <el-tag closable @click="removeRightById(scope.row, item2.id)" type="success">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable @click="removeRightById(scope.row, item3.id)" :key="item3.id" type="warning" v-for="item3 in item2.children">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size='mini' type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size='mini' type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size='mini' type="warning" icon="el-icon-set-up"
            @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%" @close="setRightDialogClosed">
<!--       树形空间-->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all
      :default-checked-keys="defaultKey"  ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolelist: [],
      rightslist: [],
      // 展示控制分配权限对话框
      setRightDialogVisible: false,
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defaultKey: [],
      roleId: ''
    }
  },
  created () {
    this.getRoleslist()
  },
  methods: {
    async getRoleslist () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rolelist = res.data
      // console.log(this.rolelist)
    },
    // 根据id删除对应权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('是否删除此权限', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      console.log(role)
      role.children = res.data
    },
    // 展示分配权限
    async showSetRightDialog (role) {
      // 获取权限数据
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightslist = res.data
      // 递归获取三级节点id
      this.getLeafKeys(role, this.defaultKey)
      this.setRightDialogVisible = true
    },
    // 通过递归获取所有权限id
    getLeafKeys (node, arr) {
      // 如果当前node不包含child则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限关闭对话框事件
    setRightDialogClosed () {
      this.defaultKey = []
    },
    // 为点击角色赋权
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.getRoleslist()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style scoped>
.el-tag{
  margin: 7px;
}

.bdtop{
  border-top: 1px solid #eee;
}

.bdbottom{
   border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
