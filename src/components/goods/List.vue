<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>
         <!-- 卡片区 -->
         <el-card>
            <!-- 搜索添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" clearable v-model="queryInfo.query"  @clear="getGoodsList">
                        <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                    </el-input> 
                </el-col>
                <el-col :span="2">
                    <el-button type="primary" @click="goAddpage">添加商品</el-button>
                </el-col>
            </el-row>
            <!-- 用户列表区 -->
            <el-table :data="goodslist" border stripe>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="商品名称" prop="goods_name"></el-table-column>
                <el-table-column label="商品价格" prop="goods_price" width="95px"></el-table-column>
                <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
                <el-table-column label="创建时间" width="140px">
                    <template slot-scope="scope">
                        {{ scope.row.add_time | dateFormat }}
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="130px">
                        <template slot-scope="scope">
                            <!-- 编辑按钮 -->
                            <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                            <!-- 删除按钮 -->
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"></el-button>
                        </template>
                </el-table-column>
            </el-table>
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
    </div>
</template>

<script>
export default{
    data() {
        return {
             //获取商品列表数据
             queryInfo: {
                query:'',
                //当前页数
                pagenum:1,
                //当前每页显示多少条数据
                pagesize:10
            },
            //商品列表
            goodslist:[],
            //总数
            total:0
        }
    },
    created() {
        this.getGoodsList()
    },
    methods: {
        //获取所有的商业列表
        async getGoodsList() {
            const { data: res } = await this.$http.get('goods',{
                 params:this.queryInfo 
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品分类失败！')
            }
            this.goodslist = res.data.goods
            this.total = res.data.total
        },
        // 监听pagesize改变的事件
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize
            this.getGoodsList()
        },
         // 监听页码值改变的事件
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage
            this.getGoodsList()
        },
         // 根据Id删除对应的参数项
        async removeById(Id) {
            const confirmResult = await this.$confirm(
                '此操作将永久删除该参数, 是否继续?',
                '提示',
                {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }
            ).catch(err => err)

            // 用户取消了删除的操作
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除！')
            }

            // 删除的业务逻辑
            const { data: res } = await this.$http.delete(
                `goods/${Id}`
            )

            if (res.meta.status !== 200) {
                return this.$message.error('删除失败！')
            }

            this.$message.success('删除成功！')
            this.getGoodsList()
        },
        //点击添加商品跳转函数
        goAddpage() {
            this.$router.push('goods/add')
        }
    }
}
</script>

<style lang="less" scoped>

</style>