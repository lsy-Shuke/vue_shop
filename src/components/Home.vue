<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>Header<el-button type="info" @click="logout">退出登录</el-button></el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px':'200px'">
        <div class="toggle-button" @click ="toggleCollapse()">III</div>
        <!-- 侧边栏菜单区域 -->
        <el-menu background-color="#2b4b6b" text-color="#ffffff" active-text-color="#1467d3" :unique-opened="true" :collapse="isCollapse" :collapse-transition="false" :router="true">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="icommonObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id">
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧主体区域 -->
      <el-main>
        <!-- 路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      // 左侧菜单数据
      menulist: [],
      icommonObj: {
        125: 'iconfont icon-user-tie',
        103: 'iconfont icon-lock',
        101: 'iconfont icon-database',
        102: 'iconfont icon-command',
        145: 'iconfont icon-stats-dots'
      },
      isCollapse: false
    }
  },
  created () {
    this.getMenuList()
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有的菜单
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    // 点击按钮，切换菜单的折叠与展开
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #ffb6c1;
} 
.el-aside {
  background-color: #2b4b6b;
  .el-menu {
    border-right: none;
  }
}
.iconfont {
  margin-right: 5px;
}
.toggle-button {
  background-color: rgb(61, 126, 212);
  font-size: 10px;
  line-height: 24px;
  color: white;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
