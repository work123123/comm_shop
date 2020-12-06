<template>
  <el-container class="home-container">
    <!-- 头部区域-->
    <el-header>
      <div>
        <img src="../assets/heima.png">
        <span>电商管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主题区域-->
    <el-container>
      <!-- 侧边栏-->
      <el-aside :width="isCollapse ? '64px': '200px'">
        <!-- 侧边栏菜单区域 -->
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#333744" text-color="#fff"
          active-text-color="#ffd04b" unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
          <!-- 一级菜单-->
          <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
            <!-- 一级菜单模板区-->
            <template slot="title">
              <!-- 图标-->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单-->
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
              <!-- 二级菜单模板区-->
              <template slot="title">
                <!-- 图标-->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{ subItem.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容主体 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>

</template>

<script>
export default {
  data () {
    // 左侧菜单数据
    return {
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      // 被激活的连接地址
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      // console.log(this.menulist)
    },
    // 点击按钮切换折叠与展开
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
    }
  }
}
</script>

<style lang="less" scoped>
.home-container{
  height: 100%;
}

.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div{
    display: flex;
    align-items: center;
    span{
      margin-left: 15px;
    }
  }
}

.el-aside{
  background-color: #333744;
  .el-menu{
    border-right: none;
  }
}

.el-main{
  background-color: #eaedf1;
}

.iconfont{
  margin-right: 10px;
}

.toggle-button{
  background-color: #4A5064;
  font-size: 10px;
  text-align: center;
  line-height: 24px;
  color: #fff;
  letter-spacing: 0.2em;
  cursor:pointer;
}
</style>
