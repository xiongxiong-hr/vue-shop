<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区 -->
        <el-card>
            <!-- 警告区 -->
            <el-alert
                title="注意：只允许为第三级分类设置相关参数!"
                :closable="false"
                show-icon
                type="warning">
            </el-alert>
            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <!-- 选择商品分类的级联选择框 -->
                    <el-cascader
                     expand-trigger="hover" 
                     :options="catelist"
                     :props="cateProps" 
                     v-model="selectedCateKeys"
                     @change="handleChange"
                    clearable
                    >
                    </el-cascader>
                </el-col>
            </el-row>
            <!-- tab标签区域 -->
            <el-tabs v-model="activeName" @tab-click="handleClick">
                <!-- 添加动态参数的面板 -->
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini"
                    :disabled="isBtnDisabled" @click="addDialogVisible = true">添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand">
                                <template slot-scope="scope">
                                    <el-tag closable @close="handleDelete(i,scope.row)" v-for="(item,i) in scope.row.attr_vals" :key="i">
                                        {{item}}
                                    </el-tag>
                                     <!-- 输入文本框 -->
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
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                                </template>
                        </el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <!-- 编辑按钮 -->
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditClick(scope.row.attr_id)">编辑</el-button>
                                <!-- 删除按钮 -->
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <!-- 添加静态属性的面板 -->
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="mini"
                    :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>
                    <!-- 静态参数面板 -->
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <!-- 循环渲染tag标签 -->
                                <el-tag closable @close="handleDelete(i,scope.row)" v-for="(item,i) in scope.row.attr_vals" :key="i">
                                    {{item}}
                                </el-tag>
                                <!-- 输入文本框 -->
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
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <!-- 编辑按钮 -->
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditClick(scope.row.attr_id)">编辑</el-button>
                                <!-- 删除按钮 -->
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
        <!-- 添加参数的对话框 -->
        <el-dialog
            :title="'修改'+titleText"
            :visible.sync="addDialogVisible"
            @close="addDialogClosed"
            width="50%">
            <!-- 内容主体区域 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button @click="addParams">确 定</el-button>
            </span>
        </el-dialog>
          <!-- 修改参数的对话框 -->
          <el-dialog
          :title="'添加'+titleText"
          :visible.sync="editDialogVisible"
          @close="editDialogClosed"
          width="50%">
          <!-- 内容主体区域 -->
          <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
              <el-form-item :label="titleText" prop="attr_name">
                  <el-input v-model="editForm.attr_name"></el-input>
              </el-form-item>
          </el-form>
          <!-- 底部区域 -->
          <span slot="footer" class="dialog-footer">
              <el-button @click="editDialogVisible = false">取 消</el-button>
              <el-button @click="editParams">确 定</el-button>
          </span>
      </el-dialog>
    </div>
</template>

<script>
    export default{
        data() {
            return {
                catelist:[],
                //级联选择框的配置对象
                cateProps:{
                    value:'cat_id',
                    label:'cat_name',
                    children:'children'
                },
                //级联选择框双向绑定
                selectedCateKeys:[],
                //被激活的标签的名称
                activeName:'many',
                //动态参数数据
                manyTableData:[],
                //静态参数数据
                onlyTableData:[],
                //显示对话框的显示与隐藏
                addDialogVisible:false,
                //添加参数的表单数据对象
                addForm:{
                    attr_name:''
                },
                //添加表单验证规则
                addFormRules:{
                    attr_name: [
                        { required: true, message: '请输入参数名称名 ', trigger:'blur'},
                    ]
                },
                //显示编辑对话框的显示与隐藏
                editDialogVisible:false,
                //编辑参数的表单数据对象
                editForm:{
                    attr_name:''
                },
                //添加编辑表单验证规则
                editFormRules:{
                    attr_name: [
                        { required: true, message: '请输入参数名称名 ', trigger:'blur'},
                    ]
                }
            }
        },
        created() {
            this.getCateList()
        },
        methods:{
            // 获取所有的商品分类
           async getCateList (){
              const{data:res} = await this.$http.get('categories')
              if(res.meta.status !== 200){
                  this.$message.error('获取商品分类失败!')
                }
              this.catelist=res.data
                console.log(this.catelist)
            },
            //级联选择框选项变化时会触发这个函数
            handleChange() {   
                this.getParamsData()
            },
            //tab标签点击事件
            handleClick() {
                this.getParamsData()
            },
            //获取参数的列表数据
            async getParamsData() {
                if(this.selectedCateKeys.length !== 3) {
                    this.selectedCateKeys=[]
                    this.onlyTableData=[]
                    this.manyTableData=[]
                    return
                }
                //根据所选分类的ID，和当前所选的面板，获取对应的参数
                const{data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{
                    params: { sel:this.activeName}
                })
                if(res.meta.status !== 200){
                  this.$message.error('获取参数列表失败!')
                }
                res.data.forEach(item => {
                    //将字符串转为数组
                    item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
                    //为文本框赋值
                    item.inputVisible = false
                    item.inputValue=""
                })
                console.log(res.data)
                if(this.activeName === 'many'){
                    this.manyTableData = res.data
                }else{
                    this.onlyTableData = res.data
                }
            },
            //增加对象表单关闭重置事件
            addDialogClosed() {
                this.$refs.addFormRef.resetFields()
            },
            //编辑表单关闭重置事件
            editDialogClosed() {
                this.$refs.editFormRef.resetFields()
            },
            //点击确定发起添加数据的请求
            addParams() {
               this.$refs.addFormRef.validate(async valid=>{
                   if(!valid) return
                   const{data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,{
                        attr_name:this.addForm.attr_name,
                         attr_sel:this.activeName
                    })
                    if(res.meta.status !== 201){
                        this.$message.error('添加参数失败!')
                    }
                    this.$message.success('添加参数成功!')
                    this.addDialogVisible = false
                    this.getParamsData()
               })
            },
            //点击按钮，展示修改对话框
            async showEditClick(attr_id) {
                // 查询当前参数的信息
                const{data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`)
                if(res.meta.status !== 200){
                    this.$message.error('获取参数信息失败!')
                }
                this.editForm = res.data
                this.editDialogVisible = true
            },
            //点击确定修改参数信息
            editParams() {
                this.$refs.editFormRef.validate(async valid=>{
                   if(!valid) return
                   const{data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{
                        attr_name:this.editForm.attr_name,
                         attr_sel:this.activeName
                    })
                    if(res.meta.status !== 200){
                        this.$message.error('修改参数失败!')
                    }
                    this.$message.success('修改参数成功!')
                    this.getParamsData()
                    this.editDialogVisible = false
               })
            },
            //点击删除参数信息
            async removeParams(attr_id) {
                const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }).catch(err => err );
                if(confirmResult !== "confirm"){
                    return this.$message.info('已取消删除')
                }
                const { data:res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
                if(res.meta.status !== 200 ){
                    return this.$message.error('删除信息失败')
                }
                this.getParamsData()
            },
            //文本框失去焦点或者是按下enter时触发
            async handleInputConfirm(row) {
                console.log(row)
               //文本框失去焦点时候，文本框长度为0 ，关闭文本框并清空输入的值
               if(row.inputValue.trim().length===0){
                   row.inputVisible = false
                   row.inputValue = ""
                   return
               }
               //如果没有return则证明输入有内容
               row.attr_vals.push(row.inputValue.trim())
               row.inputValue=""
               row.inputVisible=false
                // 需要发起请求，保存这次操作
                this.saveAttrVals(row)
            },
             // 将对 attr_vals 的操作，保存到数据库
            async saveAttrVals(row) {
                // 需要发起请求，保存这次操作
                const { data: res } = await this.$http.put(
                    `categories/${this.cateId}/attributes/${row.attr_id}`,
                    {
                    attr_name: row.attr_name,
                    attr_sel: row.attr_sel,
                    attr_vals: row.attr_vals.join(' ')
                    }
                )

                if (res.meta.status !== 200) {
                    return this.$message.error('修改参数项失败！')
                }

                this.$message.success('修改参数项成功！')
            },
            //点击newtag展示文本框
            showInput(row) {
                //让文本框自动获得焦点
                row.inputVisible = true
                //nextTick就是当页面上的函数重新被渲染之后，才会指定回调函数中的代码
                this.$nextTick(_ => {
                 this.$refs.saveTagInput.$refs.input.focus();
                });
            },
            //点击删除标签项
            handleDelete(i,row) {
                row.attr_vals.splice(i,1)
                this.saveAttrVals(row)
            }

    },
    computed:{
        //如果按钮需要禁用则返回true，否则返回false
        isBtnDisabled() {
            if(this.selectedCateKeys.length !== 3){
                return true
            }else{
                return false
            }
        },
        //当前选中的三级分类的ID
        cateId() {
            if(this.selectedCateKeys.length === 3){
                return this.selectedCateKeys[2]
            }else{
                return null
            }
        },
        //动态计算标题的文本
        titleText() {
            if(this.activeName === 'many'){
                return '动态参数'
            }else{
                return '静态属性'
            }
        }
    }
}
</script>

<style  lang="less" scoped>
    .cat_opt {
        margin:15px 0;
    }
    .el-tag{
        margin:10px;
    }
    .input-new-tag{
        width:120px;
    }
</style>