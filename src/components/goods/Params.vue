<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 警告区域 -->
      <el-alert
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
            expand-trigger="hover"
            :options="cateList"
            :props="cateProps"
            v-model="selectedCatKeys"
            @change="handleChange"
          >
          </el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisable"
            @click="addDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 动态参数的表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  @close="handelClosed(index, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <!-- 添加文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性面板 -->

        <el-tab-pane label="静态属性" name="only"
          ><el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisable"
            @click="addDialogVisible = true"
            >添加属性</el-button
          >
          <!-- 静态属性的表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  @close="handelClosed(index, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <!-- 添加文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="属性名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table></el-tab-pane
        >
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input> </el-form-item
      ></el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input> </el-form-item
      ></el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      //所有的商品分类
      cateList: [],
      //级联选择框的配置对象
      cateProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      //级联选择框双向绑定的数组
      selectedCatKeys: [],
      //tabs 选中的页签
      activeName: "many",
      //动态参数 的数据
      manyTableData: [],
      //静态参数 的数据
      onlyTableData: [],
      //控制添加对话框的显示/隐藏
      addDialogVisible: false,
      addForm: {
        attr_name: "",
      },
      //添加的验证规则
      addFormRules: {
        attr_name: [{ required: true, message: "请输入值", trigger: "blur" }],
      },
      //控制修改参数对话框的显示/隐藏
      editDialogVisible: false,
      //修改的表单数据
      editForm: {},
      //修改表单的验证规则
      editFormRules: {
        attr_name: [{ required: true, message: "请输入值", trigger: "blur" }],
      },
    };
  },
  created() {
    this.getCateList();
  },
  computed: {
    //两个添加按钮的禁用状态
    isBtnDisable() {
      if (this.selectedCatKeys.length !== 3) {
        return true;
      }
      return false;
    },
    //三级参数的ID
    catId() {
      if (this.selectedCatKeys.length === 3) {
        return this.selectedCatKeys[2];
      }
      return null;
    },
    //添加对话框的标题
    titleText() {
      if (this.activeName === "many") {
        return "动态参数";
      } else {
        return "静态属性";
      }
    },
  },

  methods: {
    //获取所有商品分类
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败！");
      }
      this.cateList = res.data;
    },
    handleChange() {
      this.getParamsData();
    },
    //页签变化时的函数
    handleTabClick() {
      this.getParamsData();
    },
    //获取动态/静态 数据
    async getParamsData() {
      //如果选中的不是三级分类，清空selectedCatKeys
      if (this.selectedCatKeys.length !== 3) {
        this.selectedCatKeys = [];
        this.manyTableData = [];
        this.onlyTableData = [];
        return;
      }
      const { data: res } = await this.$http.get(
        `categories/${this.catId}/attributes`,
        {
          params: {
            sel: this.activeName,
          },
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数列表失败！");
      }
      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : [];
        item.inputVisible = false;
        item.inputValue = "";
      });
      if (this.activeName === "many") {
        this.manyTableData = res.data;
      } else {
        this.onlyTableData = res.data;
      }
    },
    //监听添加对话框的关闭
    addDialogClosed() {
      //重置
      this.$refs.addFormRef.resetFields();
    },
    //点击按钮添加参数
    addParams() {
      //表单预验证
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          `categories/${this.catId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加失败!");
        }
        this.$message.success("添加成功!");
        this.addDialogVisible = false;
        this.getParamsData();
      });
    },
    //修改的对话框
    async showEditDialog(attr_id) {
      const { data: res } = await this.$http.get(
        `categories/${this.catId}/attributes/${attr_id}`,
        {
          params: {
            attr_sel: this.activeName,
          },
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取失败！");
      }
      this.editForm = res.data;
      console.log(this.editForm);
      this.editDialogVisible = true;
    },
    //
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    //修改参数信息
    editParams() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) {
          this.$message.error("修改失败");
        }
        const { data: res } = await this.$http.put(
          `categories/${this.catId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("修改失败");
        }
        this.$message.success("修改成功!");
        this.getParamsData();
        this.editDialogVisible = false;
      });
    },
    //根据ID删除属性
    async removeParams(attr_id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该属性, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("取消了操作");
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.catId}/attributes/${attr_id}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败!");
      }
      this.$message.success("删除成功!");
      this.getParamsData();
    },
    //展开行的文本框
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = "";
        row.inputVisible = false;
        return;
      }
      row.attr_vals.push(row.inputValue.trim());
      row.inputValue = "";
      row.inputVisible = false;
      this.saveAttrValues(row);
    },
    //
    async saveAttrValues(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.catId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(" "),
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("修改失败！");
      }
      this.$message.success("修改成功！");
    },
    //展开行的添加按钮
    showInput(row) {
      row.inputVisible = true;
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    //删除参数tag
    handelClosed(i, row) {
      row.attr_vals.splice(i, 1);
      this.saveAttrValues(row);
    },
  },
};
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 100px;
}
</style>