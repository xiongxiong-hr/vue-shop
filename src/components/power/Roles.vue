<template>
    <div>
         <!-- 面包屑导航区 -->
         <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
         <!-- 卡片区 -->
        <el-card>
            <!-- 添加角色按钮 -->
            <el-row>
                <el-col>
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表区域 -->
            <el-table :data="rolesList" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdBottom',i1===0?'bdTop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                            <!-- 一级权限 -->
                            <el-col :span="5">
                                <el-tag  
                                closable
                                @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 二级权限三级权限 -->
                            <el-col :span="19">
                                <el-row :class="[i2===0?'':'bdTop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                                    <!-- 通过for循环渲染二级权限 -->
                                    <el-col :span="6">
                                        <el-tag 
                                        closable
                                        @close="removeRightById(scope.row,item2.id)"
                                        type="success">
                                        {{item2.authName}}
                                        </el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                     <!-- 通过for循环渲染三级权限 -->
                                    <el-col :span="18">
                                        <el-tag 
                                        closable
                                        @close="removeRightById(scope.row,item3.id)"
                                        type="warning" v-for="(item3,i3) in item2.children" 
                                        :key="item3.id">
                                                {{item3.authName}}
                                        </el-tag>     
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <!-- 编辑按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                        <!-- 删除按钮 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini"></el-button>
                        <!-- 分配角色按钮 -->
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)"></el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!-- 分配权限的对话框 -->
        <el-dialog
            title="分配权限"
            :visible.sync="SetRightDialogVisible"
            width="50%"
            @close="SetRightDialoClosed"
           >
           <!-- 树形控件 -->
           <el-tree 
           show-checkbox
           :data="rightsList" 
           :props="treeProps" 
           node-key="id"
           ref="treeRef"
           default-expand-all
           :default-checked-keys="defKeys"	
           ></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="SetRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default{
    data() {
        return {
            rolesList:[],
            //控制分配权限的对话框
            SetRightDialogVisible:false,
            rightsList:[],
            treeProps: {
                children: 'children',
                label: 'authName'
            },
            // 默认选中的节点ID值
            defKeys:[],
            //当前即将分配权限的ID
            roleId:''
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        //分配权限按钮的显示
        async showSetRightDialog(role){
            this.roleId=role.id
            const {data:res} = await this.$http.get('rights/tree')
            if(res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败')
            }
            this.rightsList = res.data
            this.getLeafKeys(role,this.defKeys)
            this.SetRightDialogVisible = true;
        },
        //获取权限数据
        async getRolesList(){
            const {data:res} = await this.$http.get('roles')
            if(res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败')
            }
            this.rolesList = res.data
        },
        //根据Id删除对应的权限
        async removeRightById(role,rightId) {
            // 弹框提示用户是否要删除
            const confirmResult = await this.$confirm(
                '此操作将永久删除该文件, 是否继续?',
                '提示',
                {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }
            ).catch(err => err)
            if (confirmResult !== 'confirm') {
                return this.$message.info('取消了删除！')
            }
            const { data: res } = await this.$http.delete(
                `roles/${role.id}/rights/${rightId}`
            )

            if (res.meta.status !== 200) {
                return this.$message.error('删除权限失败！')
            }
             // this.getRolesList()
            role.children = res.data
        },
        //通过递归的方式，获取角色下所有的三级权限的id，并保存到defKeys中
        getLeafKeys(node,arr) {
            // 如果当前node节点不包含children属性，则是三级节点
            if(!node.children){
                return arr.push(node.id)
            }
            node.children.forEach(item => {
                this.getLeafKeys(item,arr)
            });
        },
        //监听分配权限对话框关闭事件
        SetRightDialoClosed(){
            this.defKeys=[]
        },
        // 分配权限的函数
        async allotRights(){
            const keys=[
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            // this.SetRightDialogVisible = false
            const idStr = keys.join(',')
            const{data:res}=await this.$http.post(`roles/${this.roleId}/rights`,{ rids: idStr })
            
            if (res.meta.status !== 200) {
                return this.$message.error('分配权限失败！')
            }

            this.$message.success('分配权限成功！')
            this.getRolesList()
            this.SetRightDialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>
    .el-tag {
        margin:7px;
    }
    .bdTop{
        border-top: 1px solid #eee;
    }
    .bdBottom{
        border-bottom: 1px solid #eee;
    }
    .vcenter{
        display: flex;
        align-items: center,
    }
</style>