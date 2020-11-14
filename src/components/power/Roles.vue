<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand" label="详细">
          <template slot-scope="scope">
            <el-row
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
              :class="['bd-bottom', 'vcenter', index1 === 0 ? 'bd-top' : '']"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二和三级权限 -->
              <el-col :span="19">
                <!-- 二级权限 -->
                <el-row
                  v-for="(item2, index2) in item1.children"
                  :key="item2.id"
                  :class="[index2 === 0 ? '' : 'bd-top', 'vcenter']"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3, index3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
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
            <el-button type="primary" icon="el-icon-edit" size="mini"
              >编辑</el-button
            >
            <el-button type="danger" icon="el-icon-delete" size="mini"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightsList(scope.row)"
              >权限分配</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightsDialogVisible"
      width="50%"
      @close="setRightsDialogClosed"
    >
      <!--树形控件  -->
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="setRightsRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      rolesList: [],
      setRightsDialogVisible: false,
      rightsList: [],
      // 树形控件
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点的ID值数组
      defKeys: [],
      // 当前选中的角色ID
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data } = await this.$http.get('roles')
      if (data.meta.status !== 200) { return this.$message.error('获取角色列表失败') }
      this.rolesList = data.data
    },
    // 根据ID删除对应的权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (data.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      // 重新获取列表会使表格全部重新渲染
      /* this.getRolesList(); */
      role.children = data.data
      this.$message.success('删除权限成功')
    },
    // 显示分配权限的对话框
    async showSetRightsList (role) {
      this.roleId = role.id
      const { data } = await this.$http.get('rights/tree')
      if (data.meta.status !== 200) {
        return this.$message.error('获取权限列表失败!')
      }
      this.rightsList = data.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightsDialogVisible = true
    },
    // 递归显示三级权限的id 放入defKeys数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach((item) => this.getLeafKeys(item, arr))
    },
    // 关闭权限对话框时，清空defKeys
    setRightsDialogClosed () {
      this.defKeys = []
    },
    // 分配权限
    async setRights () {
      const keys = [
        ...this.$refs.setRightsRef.getCheckedKeys(),
        ...this.$refs.setRightsRef.getHalfCheckedKeys()
      ]
      const keysStr = keys.join(',')
      const { data } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: keysStr
      })
      if (data.meta.status !== 200) {
        return this.$message.error('分配权限失败!')
      }
      this.getRolesList()
      this.$message.success('分配权限成功')
      this.setRightsDialogVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
.bd-top {
  border-top: 1px solid #eee;
}
.bd-bottom {
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
