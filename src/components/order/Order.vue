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
             <!-- 搜索添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容">
                        <el-button slot="append" icon="el-icon-search"></el-button>
                    </el-input> 
                </el-col>
            </el-row>
              <!-- 用户列表区 -->
            <el-table :data="orderslist" border stripe>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="订单编号" prop="order_number"></el-table-column>
                <el-table-column label="订单价格" prop="order_price" width="95px"></el-table-column>
                <el-table-column label="是否付款" width="90px">
                        <template slot-scope="scope">
                            <el-tag v-if="scope.row.order_pay==1" type="success" size="mini">已付款</el-tag>
                            <el-tag v-else type="danger" size="mini">未付款</el-tag>
                        </template>
                </el-table-column>
                <el-table-column label="是否发货" prop="is_send" width="95px"></el-table-column>
                <el-table-column label="下单时间" width="150px">
                    <template slot-scope="scope">
                       {{ scope.row.update_time | dateFormat }}
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="130px">
                    <template slot-scope="scope">
                        <!-- 编辑按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox"></el-button>
                        <!-- 删除按钮 -->
                        <el-button type="success" icon="el-icon-location" size="mini" @click="showProgressBox"></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 页码区 -->
            <el-pagination
                background
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[5,10,15]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
          </el-pagination>
        </el-card>
        <!-- 修改地址的对话框 -->
        <el-dialog
            title="修改地址"
            :visible.sync="addressVisible"
            @close="addressDialogClosed"
            width="50%">
            <!-- 内容主体区域 -->
            <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
                <el-form-item label="省市区/县" prop="address1">
                    <!-- 级联选择器 -->
                    <el-cascader
                        :options="cityData"
                        v-model="addressForm.address1">
                    </el-cascader>
                </el-form-item>
                <el-form-item label="详细地址" prop="address2">
                    <el-input v-model="addressForm.address2"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addressVisible = false">取 消</el-button>
                <el-button @click="addressVisible = false">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 展示物流进度的对话框 -->
        <el-dialog
            title="物流信息"
            :visible.sync="ProgressVisible"
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
    import cityData from './citydata.js'
    export default{
       data() {
           return{
                //获取商品列表数据
                queryInfo: {
                    query:'',
                    //当前页数
                    pagenum:1,
                    //当前每页显示多少条数据
                    pagesize:10
                },
                //订单列表
                orderslist:[],
                //修改地址
                addressVisible:false,
                addressForm: {
                    address1:[],
                    address2:''
                },
                addressFormRules:{
                    address1: [
                         { required: true, message: '请输入省市区/县', trigger:'blur'}
                   ],
                   address2: [
                     { required: true, message: '请输入详细地址', trigger:'blur'}
                   ]
                },
                //显示物流对话框
                ProgressVisible:false,
                cityData,
                //物流进度数据
                progressInfo:[]
           }
       },
       created() {
           this.getOrderList()
       },
       methods:{
            //获取所有的商业列表
            async getOrderList() {
                const { data: res } = await this.$http.get('orders',{
                 params:this.queryInfo 
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取订单列表失败！')
            }
                this.orderslist = res.data.goods
                this.total = res.data.total
                console.log(res.data)
            },
            // 监听pagesize改变的事件
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize
                this.getOrderList()
            },
            // 监听页码值改变的事件
            handleCurrentChange(newPage) {
                this.queryInfo.pagenum = newPage
                this.getOrderList()
            },
            //展示修改地址的对话框
            showBox() {
                this.addressVisible = true
            },
            addressDialogClosed() {
                this.$refs.addressFormRef.resetFields()
            },
            //查看物流进度函数
            async showProgressBox() {
                //获取物流信息数据
                const { data: res } = await this.$http.get('/kuaidi/1106975712662')
                if (res.meta.status !== 200) {
                    return this.$message.error('获取物流信息失败！')
                }
                this.progressInfo = res.data
                console.log(this.progressInfo)
                this.ProgressVisible = true
            }
        }
    }
</script>

<style lang="less" scoped>
    .el-cascader {
        width:100%;
    }
</style>