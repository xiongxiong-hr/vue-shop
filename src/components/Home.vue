<template>
    <el-container class="home-container">
        <!-- 头部区域 -->
        <el-header>
            <div>
                <img src="../assets/heima.png" alt=""> 
                <span>电商后台管理系统</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <!-- 页面主体区 -->
        <el-container>
            <!-- 侧边栏 -->
            <el-aside :width="isCollapse ? '64px' : '200px'">
                    <div class="toggle-button" @click="toggleCollapse">|||</div>
                    <!-- 侧边栏 菜单区域-->
                    <el-menu
                        background-color="#333744"
                        text-color="#fff"
                        unique-opened
                        :collapse="isCollapse"
                        router
                        :collapse-transition="false"
                        :default-active="activePath"
                        active-text-color="#409EFF">
                        <!-- 一级菜单 -->
                        <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
                            <!-- 一级菜单模板区域 -->
                            <template slot="title">
                                <i :class="iconsObj[item.id]"></i>
                                <!-- 文本 -->
                                <span>{{item.authName}}</span>
                            </template>
                             <!-- 二级菜单 -->
                            <el-menu-item 
                             :index="'/'+subItem.path+''" 
                             v-for="subItem in item.children"
                             @click="saveNavState('/'+subItem.path)"
                             :key="subItem.id">
                                    <template slot="title">
                                        <i class="el-icon-menu"></i>
                                        <!-- 文本 -->
                                        <span>{{subItem.authName}}</span>
                                    </template>
                            </el-menu-item>
                        </el-submenu>
                </el-menu>
            </el-aside>
            <!-- 右侧内容主题 -->
            <el-main>
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
export default {
    data() {
        return {
            menulist:[],
            iconsObj:{
                '125':'iconfont icon-user',
                '103':'iconfont icon-tijikongjian',
                '101':'iconfont icon-shangpin',
                '102':'iconfont icon-danju',
                '145':'iconfont icon-baobiao',
            },
            isCollapse:false,
            activePath:''
        }
    },
    created() {
        this.getMenuList(),
        this.activePath=window.sessionStorage.getItem('activePath')
    },
    methods: {
        logout() {
            window.sessionStorage.clear();
            this.$router.push("/login");
        },
        async getMenuList() {
           const{data:res} = await this.$http.get('menus')
           if(res.meta.status !== 200) return this.$massage.error(res.meta.msg)
           this.menulist=res.data
           console.log(this.menulist)
        },
        //点击按钮，切换状态
        toggleCollapse() {
            this.isCollapse = !this.isCollapse
        },
        saveNavState(activePath) {
            window.sessionStorage.setItem('activePath',activePath)
            this.activePath=activePath
        }
    }
};
</script>

<style lang="less" scoped>
  .home-container {
      height:100%
  }  
 .el-header {
     background-color:#373d41;
     display:flex;
     justify-content: space-between;
     align-items: center;
     color:#fff;
     padding-left:0;
     > div{
         display: flex;
         align-items:center;
         span{
             margin-left:15px;
         }
     }
     
 }
 .el-aside {
     background-color:#333744;
     .el-menu{
         border-right:none;
    }
 }
 .el-main {
    background-color:#eaedf1;
 }
 .iconfont{
     margin-right:10px;
 }
 .toggle-button{
     background-color: #4a5064;
     line-height:24px;
     font-size: 10px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
 }
</style>