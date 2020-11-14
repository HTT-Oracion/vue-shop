<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <!-- 添加分类 按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCate">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table
        class="treeTable"
        :data="cateList"
        :columns="columns"
        show-index
        index-text="#"
        border
        :expand-type="false"
        :selection-type="false"
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: green"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            type="success"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
    >
      <!-- 添加分类的表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFromRules"
        ref="addCateFromRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options 用来指定数据源  -->
          <el-cascader
            v-model="selectKeys"
            expand-trigger="hover"
            :options="parentCateList"
            :props="cascaderList"
            @change="parentCateChange"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCateDialogVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //商品分类的列表
      cateList: [],
      //总数据条数
      total: 0,
      //查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      //控制添加分类对话框的显示
      addCateDialogVisible: false,
      //添加分类的表单数据
      addCateForm: {
        //添加分类的 名称
        cat_name: "",
        //分类父 ID
        cat_pid: 0,
        //分类等级
        cat_level: 0,
      },
      //添加分类表单的 规则
      addCateFromRules: {
        cat_name: [
          {
            required: true,
            message: "请输入分类名称",
            trigger: "blur",
          },
        ],
      },
      //父级分类的列表
      parentCateList: [],
      //级联选择器
      cascaderList: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      //选中的父级分类的ID数组
      selectKeys: [],
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        //自定义模板列
        {
          label: "是否有效",
          type: "template",
          template: "isok",
        },
        {
          label: "排序",
          type: "template",
          template: "order",
        },
        {
          label: "操作",
          type: "template",
          template: "opt",
        },
      ],
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (data.meta.status !== 200) {
        return this.$message.error("获取商品分类师表");
      }
      this.cateList = data.data.result;
      this.total = data.data.total;
    },
    //监听 pagesize
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize;
      this.getCateList();
    },
    //监听当前页码值
    handleCurrentChange(newpage) {
      this.queryInfo.pagenum = newpage;
      this.getCateList();
    },
    //显示添加分类的对话框
    showAddCate() {
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    async getParentCateList() {
      const { data } = await this.$http.get("categories", {
        params: {
          type: 2,
        },
      });
      if (data.meta.status !== 200) {
        this.$message.error("获取失败!");
      }
      this.parentCateList = data.data;
    },
    //级联选择器选项变化
    parentCateChange() {
      console.log(this.selectKeys);
    },
  },
};
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>