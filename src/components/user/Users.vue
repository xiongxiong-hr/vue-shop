<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
         </el-breadcrumb>
         <!-- 卡片区 -->
         <el-card>
             <!-- 搜索添加区域 -->
             <el-row :gutter="20">
                 <el-col :span="8">
                        <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                            <el-button slot="append" icon="el-icon-search" @click="getUserList">
                            </el-button>
                        </el-input> 
                 </el-col>
                 <el-col :span="2">
                     <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
                 </el-col>
             </el-row>
             <!-- 用户列表区 -->
             <el-table :data="userList" border stripe>
                 <el-table-column type="index" label="#"></el-table-column>
                 <el-table-column label="姓名" prop="username"></el-table-column>
                 <el-table-column label="邮箱" prop="email"></el-table-column>
                 <el-table-column label="电话" prop="mobile"></el-table-column>
                 <el-table-column label="角色" prop="role_name"></el-table-column>
                 <el-table-column label="状态" prop="mg_state">
                     <template slot-scope="scope">
                         <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
                     </template>
                 </el-table-column>
                 <el-table-column label="操作" width="180px">
                        <template slot-scope="scope">
                            <!-- 编辑按钮 -->
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                            <!-- 删除按钮 -->
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                            <!-- 分配角色按钮 -->
                            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                                    <el-button type="warning" icon="el-icon-setting" size="mini"  @click="setRole(scope.row)"></el-button>
                            </el-tooltip>
                        </template>
                 </el-table-column>
             </el-table>
             <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 3, 6]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
             </el-pagination>
        </el-card>
         <!-- 添加用户信息对话框 --> 
        <el-dialog
            title="添加用户"
            :visible.sync="addDialogVisible"
            @close="addDialogClosed"
            width="50%">
            <!-- 内容主体区域 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
       </el-dialog>
       <!-- 编辑用户信息对话框 -->
       <el-dialog
        title="修改用户"
        @close="editDialogClosed"
        :visible.sync="editDialogVisible"
        width="70%">
         <!-- 内容主体区域 -->
         <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo">确 定</el-button>
        </span>
        </el-dialog>
        <!-- 分配角色的对话框 -->
        <el-dialog
        title="分配角色"
        @close="setRoleDialogClosed"
        :visible.sync="setRoleDialogVisible"
        width="50%">
            <div>
                <p>当前用户: {{userinfo.username}}</p>
                <p>当前角色: {{userinfo.role_name}}</p>
                <p>分配新角色:
                    <el-select v-model="selectedRoleId" placeholder="请选择">
                        <el-option
                          v-for="item in rolesList"
                          :key="item.id"
                          :label="item.roleName"
                          :value="item.id">
                        </el-option>
                      </el-select>
                </p>
            </div>
           
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
            </span>
        </el-dialog>
    </div>
   
</template>

<script>
export default{
    data() {
        //邮箱验证规则
        var checkEmail = (rule,value,cb) => {
            //定义一个验证邮箱的正则
            const regEmail = /^[a-zA-Z0-9_-]+@([a-zA-Z0-9_-])+(\.[a-zA-z0-9_-]+)/
            if(regEmail.test(value)){
                //合法的邮箱
                return cb()
            }
            cb(new Error('请输入合法的邮箱'))
        }
            //手机号验证规则
        var checkMobile = (rule,value,cb) => {
            // 定义一个验证手机号的正则
            const regMobile = /^(0|86|17951)?(13[0-9]|15[0-9]|17[678]|18[0-9]|14[57])[0-9]{8}$/
            if(regMobile.test(value)){
                //合法的邮箱
                return cb()
            }
            cb(new Error('请输入合法的手机'))
        }
        return{
            //获取用户登录参数
            queryInfo: {
                query:'',
                //当前页数
                pagenum:1,
                //当前每页显示多少条数据
                pagesize:2
            },
            userList: [],
            //所有角色数据列表
            rolesList:[],
            total: 0,
            // 控制添加对话框的显示与隐藏
            addDialogVisible:false,
            //添加用户表单数据
            addForm:{
                username:'',
                password:'',
                email:'',
                mobile:''
            },
            editDialogVisible:false,
            //分配角色对话框的显示与隐藏
            setRoleDialogVisible:false,
            editForm:{},
            //需要被分配角色的用户信息
            userinfo:{},
            //已选中的角色Id值
            selectedRoleId:'',
            //添加用户验证规则
            addFormRules: {
                username: [
                    { required: true, message: '请输入用户名 ', trigger:'blur'},
                    { min: 3,max: 10, message:'用户名的长度3-10个之间', trigger:'blur'}
                ],
                password: [
                    { required: true, message: '请输入密码 ', trigger:'blur'},
                    { min: 6,max: 15, message:'用户名的长度6-15个之间', trigger:'blur'}
                ],
                email: [
                    { required: true, message: '请输入邮箱', trigger:'blur'},
                    { validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    { required: true, message: '请输入电话号码', trigger:'blur'},
                    { validator: checkMobile, trigger: 'blur' }
                ]
            },
            editFormRules:{
                email: [
                    { required: true, message: '请输入邮箱', trigger:'blur'},
                    { validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    { required: true, message: '请输入电话号码', trigger:'blur'},
                    { validator: checkMobile, trigger: 'blur' }
                ]
            }
        }
    },
    created() {
        this.getUserList();
    },
    methods: {
        async getUserList(){
           const {data:res} = await this.$http.get('users',{ params:this.queryInfo })
           if(res.meta.status !== 200){return this.$message.error('获取用户数据列表失败')};
           this.userList = res.data.users
           this.total = res.data.total
        },
        // 监听pagesize改变的事件
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize
            this.getUserList()
        },
         // 监听页码值改变的事件
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage
            this.getUserList()
        },
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },
        editDialogClosed(){
            this.$refs.editFormRef.resetFields()
        },
        //添加用户按钮
        addUser(){
            this.$refs.addFormRef.validate(async valid=>{
                if(!valid) return
                //发起添加用户的请求
                const {data:res} = await this.$http.post('users',this.addForm)
                if(res.meta.status !== 201) {this.$message.error('添加用户成失败!')}
                this.$message.success('添加用户成功')
                this.addDialogVisible=false
                this.getUserList()
            })
        },
        //编辑用户按钮
        editUserInfo(){
            this.$refs.editFormRef.validate(async valid =>{
                if(!valid) return
                //发起修改数据请求
                const {data:res} = await this.$http.put('users/'+this.editForm.id,{
                    email:this.editForm.email,
                    mobile:this.editForm.mobile
                })
                if(res.meta.status !== 200) { return this.$message.error('修改用户数据失败!')}
              
                this.editDialogVisible=false
                this.getUserList()
                this.$message.success('修改用户成功')
            })
        },
        //查询用户信息
       async showEditDialog(id){
          const {data:res} = await this.$http.get('users/'+id)
          if(res.meta.status !== 200) {this.$message.error('查询用户信息失败')}
          this.editForm = res.data
          this.editDialogVisible=true
        },
        //监听开关状态的改变
        async userStateChanged(userinfo) {
         const {data:res }= await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
            console.log(res)
            if (res.meta.status !== 200) {
                userinfo.mg_state = !userinfo.mg_state
                return this.$message.error('更新用户状态失败')
            }
            this.$message.success('更新用户状态成功')
        },
        //删除文本
       async removeUserById(id) {
            const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err );
            if(confirmResult !== "confirm"){
                return this.$message.info('已取消删除')
            }
            const { data:res } = await this.$http.delete('users/'+id)
            console.log(res)
            if(res.meta.status !== 200 ){
                return this.$message.error('删除信息失败')
            }
            this.getUserList()
        },
        //分配角色按钮
        async setRole(userinfo) {
            this.userinfo = userinfo
           const{data:res} = await this.$http.get('roles')
           if(res.meta.status !== 200 ){
                return this.$message.error('获取角色信息失败')
            }
            this.rolesList = res.data
            this.setRoleDialogVisible=true;

        },
        //点击确定按钮分配角色
        async saveRoleInfo() {
            if(!this.selectedRoleId){
                return this.$message.error('请选择所要分配的角色')
            }
            const{data:res} = await this.$http.put(`users/${this.userinfo.id}/role`,
                {rid: this.selectedRoleId}
            )
            if(res.meta.status !== 200 ){
                return this.$message.error('更新角色失败')
            }
            this.$message.success('更新角色成功')
            this.getUserList()
            this.setRoleDialogVisible = false
        },
        //监听分配角色对话框关闭事件
        setRoleDialogClosed() {
            this.selectedRoleId = ""
            this.userinfo = ""
        }
    }
}
</script>

<style lang="less" scoped>

</style>