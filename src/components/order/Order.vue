<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <!-- 搜索框 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" class="input-with-select">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单列表 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column
          label="订单价格"
          prop="order_price"
          width="100px"
        ></el-table-column>
        <el-table-column label="是否付款" prop="pay_status" width="100px">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status === '0'" type="danger"
              >未付款</el-tag
            >
            <el-tag v-else type="success">已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="是否发货"
          prop="is_send"
          width="100px"
        ></el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="120px">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showBox"
            ></el-button>
            <el-button
              size="mini"
              type="success"
              icon="el-icon-location"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分页器 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>
    <!-- 修改地址的对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressDialogVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <el-form
        :model="addressForm"
        :rules="addressFormrRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item prop="address1" label="省市区/县">
          <el-cascader :options="cityData" v-model="addressForm.address1">
          </el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressDialogVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 物流进度的对话框 -->
    <el-dialog
      title="物流进度"
      :visible.sync="progressDialogVisible"
      width="50%"
    >
      <span> 物流信息接口有问题，暂不写这个功能</span>
    </el-dialog>
  </div>
</template>
<script>
import cityData from "./citydata";
export default {
  data() {
    return {
      //查询对象
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      //总数
      total: 0,
      //订单列表
      orderList: [],
      //修改地址的对话框
      addressDialogVisible: false,
      progressDialogVisible: false,
      //
      addressForm: {
        address1: [],
        address2: "",
      },
      //验证规则
      addressFormrRules: {
        address1: [
          {
            required: true,
            message: "请选择省市区",
            trigger: "blur",
          },
        ],
        address2: [
          {
            required: true,
            message: "请填写地址",
            trigger: "blur",
          },
        ],
      },
      //省市区
      cityData,
      //快递进度
      progressList: [],
    };
  },
  created() {
    this.getOrderList();
  },
  methods: {
    //获取订单列表
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取订单失败!");
      }
      this.total = res.data.total;
      this.orderList = res.data.goods;
    },
    //分页器
    //控制pagesize
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getOrderList();
    },
    //页码值变化
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getOrderList();
    },
    //展示修改地址的对话框
    showBox() {
      this.addressDialogVisible = true;
    },
    //关闭对话框
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields();
      this.addressDialogVisible = false;
    },
    //显示物流进度的对话框
    async showProgressBox() {
      this.progressDialogVisible = true;
    },
  },
};
</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>