<template>
        <div>
          <!-- 面包屑导航区 -->
            <el-breadcrumb separator-class="el-icon-arrow-right">
                <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
                <el-breadcrumb-item>商品管理</el-breadcrumb-item>
                <el-breadcrumb-item>添加商品</el-breadcrumb-item>
            </el-breadcrumb>
            <!-- 卡片区 -->
            <el-card>
                <!-- 提示区 -->
                <el-alert
                    title="添加商品信息"
                    center
                    :closable="false"
                    type="info"
                    show-icon>
                </el-alert>
                <!-- 步骤条区 -->
                <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
                    <el-step title="基本信息"></el-step>
                    <el-step title="商品参数"></el-step>
                    <el-step title="商品属性"></el-step>
                    <el-step title="商品图片"></el-step>
                    <el-step title="商品内容"></el-step>
                    <el-step title="完成"></el-step>
                </el-steps>
                <!-- 列表栏 -->
                <el-form 
                    :model="addForm" 
                    :rules="addFormRules" 
                    ref="addFormRef" 
                    label-width="100px" label-position="top">
                    <el-tabs 
                        :tab-position="'left'" 
                        v-model="activeIndex" 
                        @tab-click="tabClicked"
                        :before-leave="beforeTableLeave">
                        <el-tab-pane label="基本信息" name="0">
                            <el-form-item label="商品名称" prop="goods_name">
                                <el-input v-model="addForm.goods_name"></el-input>
                            </el-form-item>
                            <el-form-item label="商品价格" prop="goods_price">
                                    <el-input v-model="addForm.goods_price" type="number"></el-input>
                                </el-form-item>
                            <el-form-item label="商品重量" prop="goods_weight">
                                <el-input v-model="addForm.goods_weight" type="number"></el-input>
                            </el-form-item>
                                <el-form-item label="商品数量" prop="goods_number">
                                    <el-input v-model="addForm.goods_number" type="number"></el-input>
                                </el-form-item>
                                <el-form-item label="商品分类" prop="goods_cat">
                                    <el-cascader expand-trigger="hover" 
                                        :options="catelist"
                                        :props="cateProps" 
                                        v-model="addForm.goods_cat"
                                        @change="handleChange"
                                        clearable
                                        >
                                    </el-cascader>
                                </el-form-item>
                        </el-tab-pane>
                        <el-tab-pane label="商品参数" name="1">
                            <!-- 渲染表单的Item项 -->
                            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
                                <!-- 复选框组 -->
                                <el-checkbox-group v-model="item.attr_vals">
                                    <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border></el-checkbox>
                                </el-checkbox-group>
                            </el-form-item>
                        </el-tab-pane>
                        <el-tab-pane label="商品属性" name="2">
                            <!-- 渲染表单的Item项 -->
                            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
                                <el-input v-model="item.attr_vals"></el-input>
                            </el-form-item>
                        </el-tab-pane>
                        <el-tab-pane label="商品图片" name="3">
                             <!-- action表示图片要上传的地址 -->
                            <el-upload
                                :headers="headerObj"
                                :on-success="handleSuccess"
                                :on-remove="handleRemove"
                                :action="uploadURL"
                                :on-preview="handlePreview"
                                list-type="picture">
                                <el-button size="small" type="primary">点击上传</el-button>
                            </el-upload>
                        </el-tab-pane>
                        <el-tab-pane label="商品内容" name="4">
                            <quill-editor
                                v-model="addForm.goods_introduce"
                            ></quill-editor>
                            <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
                        </el-tab-pane>
                    </el-tabs>
                </el-form>
            </el-card>
            <!-- 图片预览dialog -->
            <el-dialog
                title="图片预览"
                :visible.sync="previewVisible"
                width="50%"
                >
                <img :src="previewPath" alt="">
            </el-dialog>
        </div>
    </template>
    
    <script>
    import _ from 'lodash'
    export default{
        data() {
            return{
                activeIndex:'0',
                //添加表单数据绑定
                addForm:{
                    goods_name: '',
                    goods_price: 0,
                    goods_weight: 0,
                    goods_number: 0,
                    goods_cat:[],
                     //拼接图片信息
                    pics:[],
                    //商品的详情描述
                    goods_introduce:'',
                    attrs:[]
                },
                //添加表单的验证规则
                addFormRules:{
                    goods_name: [
                        { required: true, message: '请输入商品名称', trigger:'blur'}
                    ],
                    goods_price: [
                        { required: true, message: '请输入商品价格', trigger:'blur'}
                    ],
                    goods_weight: [
                        { required: true, message: '请输入商品重量', trigger:'blur'}
                    ],
                    goods_number: [
                        { required: true, message: '请输入商品数量', trigger:'blur'}
                    ],
                    goods_cat: [
                        { required: true, message: '请输入商品数量', trigger:'blur'}
                    ]
                },
                //商品分类列表
                catelist: [],
                  // 指定级联选择器的配置对象
                cateProps: {
                    value: 'cat_id',
                    label: 'cat_name',
                    children: 'children'
                },
                 //动态参数数据
                 manyTableData:[],
                  //静态参数数据
                 onlyTableData:[],
                 //图片上传的地址
                 uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
                // 图片上传组件的headers请求头对象
                headerObj: {
                    Authorization: window.sessionStorage.getItem('token')
                },
               //预览图片的地址
               previewPath:'',
               //对话框的显示与隐藏
               previewVisible:false,
               
            }
        },
        created() {
            this.getCateList()
        },
        methods: {
            //获取所有商品分类数据
            async getCateList() {
                const { data: res } = await this.$http.get('categories')
                if (res.meta.status !== 200) {
                    return this.$message.error('获取商品分类失败！')
                }
                this.catelist = res.data
            },
            //级联选择器变化会触发的函数
            handleChange() {
                console.log(this.addForm.goods_cat)
                if(this.addForm.goods_cat.length !== 3){
                    this.addForm.goods_cat = []
                }
            },
            //切换tab标签触发
            beforeTableLeave(activeName,oldActiveName) {
                if( oldActiveName ==='0' && this.addForm.goods_cat.length !== 3){
                    this.$message.error('请先选择商品分类！')
                    return false
                }
            },
            async tabClicked() {
                //获取动态参数
                    if(this.activeIndex === '1'){
                        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,{
                            params: { sel:'many' }
                        })
                        if (res.meta.status !== 200) {
                            return this.$message.error('获取动态参数失败！')
                        }
                        res.data.forEach(item => {
                            //将字符串转为数组
                            item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
                        })
                         //动态参数数据
                         this.manyTableData = res.data
                         //console.log(this.manyTableData)         
                    }else if(this.activeIndex === '2'){
                         const { data: res } = await this.$http.get(
                            `categories/${this.cateId}/attributes`,
                            {
                                params: { sel: 'only' }
                            }
                        )

                        if (res.meta.status !== 200) {
                        return this.$message.error('获取静态属性失败！')
                        }

                        console.log(res.data)
                        this.onlyTableData = res.data
                                        
                    }
                },
                //图片预览的函数
                handlePreview(file) {
                    console.log(file);
                    this.previewPath = file.response.data.url
                    this.previewVisible = true
                 },
                //图片上传成功的钩子
                handleSuccess(response) {
                    // 1. 拼接得到一个图片信息对象
                    const picInfo = { pic: response.data.tmp_path }
                    // 2. 将图片信息对象，push 到pics数组中
                    this.addForm.pics.push(picInfo)  
                },
                //移除图片的钩子
                handleRemove(file) {
                    // 获取所要删除的图片路径
                    const filePath = file.response.data.tmp_path
                    //从pics数组中找到并删除这个图片的索引
                    const i= this.addForm.pics.findIndex( x => x.pic === filePath)
                    //调用数组的splice的方法，把图片信息对象，从pics数组中删除
                    this.addForm.pics.splice(i,1)
                },
                //添加商品
                add() {
                this.$refs.addFormRef.validate(async valid=>{
                   if(!valid) {
                       return this.$message.error('请填写必要的表单项')
                   }
                   // 执行添加的业务逻辑
                    // lodash   cloneDeep(obj)
                    const form = _.cloneDeep(this.addForm)
                    form.goods_cat = form.goods_cat.join(',')
                    this.manyTableData.forEach(item => {
                        const newInfo = {attr_id:item.attr_id,
                        attr_value:item.attr_vals.join(' ')}
                        this.addForm.attrs.push(newInfo)
                    })
                    this.onlyTableData.forEach(item => {
                        const newInfo = {attr_id:item.attr_id,
                        attr_value:item.attr_vals}
                        this.addForm.attrs.push(newInfo)
                    })
                    form.attrs = this.addForm.attrs
                    console.log(form)
                    //发起请求，添加商品，商品名称必须是唯一的
                    const { data: res } = await this.$http.post('goods',form)
                    if (res.meta.status !== 201) {
                            return this.$message.error('添加商品失败！')
                        }
                    this.$message.success('商品添加成功')
                    this.$router.push('/goods')
                })
              }
            },
            computed: {
                cateId() {
                    if(this.addForm.goods_cat.length === 3){
                        return this.addForm.goods_cat[2]
                    }
                    return null
                }
            }
    }
    </script>
    
    <style lang="less" scoped>
        .el-checkbox {
            margin:0 10px 0 0 !important;
        }
        .btnAdd {
            margin-top:15px;
        }
    </style>