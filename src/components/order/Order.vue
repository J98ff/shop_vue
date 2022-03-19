<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            clearable
            v-model="queryInfo.query"
            @clear="getOrderList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getOrderList"
            ></el-button
          ></el-input>
        </el-col>
      </el-row>
      <!-- 订单列表区域 -->
      <el-table :data="orderslist" border stripe>
        <!-- 索引列 -->
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column
          label="订单价格"
          prop="order_price"
          width="95px"
        ></el-table-column>
        <el-table-column
          label="是否支付"
          prop="pay_status"
          width="90px"
        >
        <template slot-scope="scope">
          <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
          <el-tag type="danger" v-else>未付款</el-tag>
        </template>
        </el-table-column>
        <el-table-column
          label="是否发货"
          prop="is_send"
          width="150px"
        ></el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="150px">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template>
            <!-- 修改地址按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showBox"
            ></el-button>
            <!-- 物流按钮 -->
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgessBox"
            >
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 修改地址的对话框 -->
    <el-dialog
  title="修改地址"
  :visible.sync="addressVisible"
  width="50%" @close="addressDialogClosed">
  <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
  <el-form-item label="省市区/县" prop="address1">
    <el-cascader :options="cityData" v-model="addressForm.address1" :props="{ expandTrigger: 'hover' }"></el-cascader>
  </el-form-item>
  <el-form-item label="详细地址" prop="adress2">
    <el-input v-model="addressForm.adress2"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addressVisible = false">取 消</el-button>
    <el-button type="primary" @click="addressVisible = false">确 定</el-button>
  </span>
</el-dialog>
<!-- 显示物流信息的对话框 -->
<el-dialog
  title="物流信息"
  :visible.sync="progressVisible"
  width="50%">
  <!-- 时间线 -->
  <el-timeline>
    <el-timeline-item
      v-for="(activity, index) in progressInfo"
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
</el-dialog>
  </div>
</template>
<script>
import cityData from './citydata'
export default {
  data () {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      orderslist: [],
      total: 0,
      addressVisible: false,
      addressForm: {
        adress1: [],
        adress2: ''
      },
      addressFormRules: {
        adress1: [{ required: true, message: '请选择省市区/县', trigger: 'blur' }],
        adress2: [{ required: true, message: '请输入详细地址', trigger: 'blur' }]
      },
      cityData,
      progressVisible: false,
      progressInfo: []
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }
      this.orderslist = res.data.goods
      this.total = res.data.total
    },
    // 监听 pagesize改变的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    // 监听页码值改变的事件
    handleCurrentChange  (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 展示修改地址的对话框
    showBox () {
      this.addressVisible = true
    },
    addressDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgessBox () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('查询物流信息失败')
      }
      this.progressInfo = res.data
      console.log(this.progressInfo)
      this.progressVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader{
  width: 100%;
}
</style>
