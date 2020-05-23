<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/logo.png" alt />
      </div>
      <span>后台电商管理系统</span>
      <el-button class="out" type="info" @click="out">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="active?'100px':'200px'">
        <div @click="active = !active" class="collspace">
          <i class="el-icon-s-fold"></i>
        </div>
        <el-menu
          unique-opened
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409eff"
          :collapse="active"
          :collapse-transition="false"
          router
          :default-active="defaultActive"
        >
          <!-- 一级菜单 -->
          <el-submenu :index="item.menu_id+''" v-for="(item,index) in menuList" :key="item.menu_id">
            <template slot="title">
              <i :class="icons[index]"></i>
              <span>{{item.menu_name}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+list.path" v-for="list in item.chirden" :key="list.menu_id">
              <i class="el-icon-menu"></i>
              <span @click="savePath('/'+list.path)">{{list.menu_name}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
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
      //刷新页面时的高亮显示
      defaultActive: '',
      //是否折叠
      active: false,
      menuList: [],
      icons: [
        'el-icon-user-solid',
        'el-icon-s-goods',
        'el-icon-s-shop',
        'el-icon-s-flag',
        'el-icon-s-operation'
      ]
    }
  },
  created() {
    this.getMenulist()
    this.getPath()
  },
  methods: {
    out() {
      window.sessionStorage.removeItem('token')
      this.$message.success('退出成功')
      this.$router.replace('/login')
    },
    async getMenulist() {
      var list = await this.$axios.get('users/menu')
      this.menuList = list.data
    },
    //保存点击后的path
    savePath(val) {
      window.sessionStorage.setItem('path', val)
    },
    //刷新页面后获取path用户高亮显示
    getPath() {
      this.defaultActive = window.sessionStorage.getItem('path')
    }
  }
}
</script>
<style lang="less" scoped>
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  div {
    width: 58px;
    height: 58px;
    border: 1px solid white;
    border-radius: 50%;
    box-shadow: 0 0 10px #ddd;
    img {
      width: 90%;
      height: 90%;
      padding: 5px;
      border-radius: 50%;
    }
  }
  span {
    font-size: 40px;
    color: white;
    margin-left: 80px;
  }
  .out {
    height: 80%;
  }
}
.el-aside {
  background-color: #333744;
  .collspace {
    width: 100%;
    height: 30px;
    background-color: ghostwhite;
    text-align: center;
  }
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.home-container {
  height: 100%;
}
</style>