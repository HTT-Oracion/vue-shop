<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button></el-header
    >
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleMeun">|||</div>
        <el-menu
          background-color="#323744"
          text-color="#fff"
          active-text-color="#369bff"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id + ''"
            v-for="item in menuList"
            :key="item.id"
          >
            <template slot="title" class="submeun-title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + subitem.path"
              v-for="subitem in item.children"
              :key="subitem.id"
              @click="changeActiveItem('/' + subitem.path)"
              ><template slot="title">
                <!-- 二级菜单图标 -->
                <i class="el-icon-menu"></i>
                <!-- 二级菜单文本 -->
                <span>{{ subitem.authName }}</span>
              </template></el-menu-item
            >
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <!-- 路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  data () {
    return {
      menuList: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data } = await this.$http.get('menus')
      if (data.meta.status !== 200) { return this.$message.error('获取菜单数据失败') }
      this.menuList = data.data
    },
    toggleMeun () {
      this.isCollapse = !this.isCollapse
    },
    changeActiveItem (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: rgba(55, 60, 65);
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 0;
  font-size: 20px;
  color: #fff;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  text-align: center;
  background-color: rgba(50, 55, 68);
  color: #fff;
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: rgba(234, 237, 242);
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  height: 20px;
  line-height: 20px;
  background-color: #485164;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
