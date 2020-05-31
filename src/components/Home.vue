<template>
  <el-container class="home_container">
    <el-header>
      <div class="class1">
        <img src="../assets/heima.png">
        <span class="span">电商后台管理系统</span>
      </div>
      <el-button type='info' @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse?'64px':'200px'">
        <div class="toggle_button" @click="toggleCollapse">|||</div>
        <el-menu :default-active="activePath" class="el-menu-vertical-demo" @open="handleOpen" @close="handleClose"
          background-color="#333744" text-color="#fff" active-text-color="#409eff" :collapse="isCollapse"
          :collapse-transition="false" :router="true" :unique-opened="true">
          <el-submenu :index="'/'+item.path" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/'+item1.path" v-for="item1 in item.children" :key="item1.id"
              @click="saveNavState('/'+item1.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{item1.authName}}</span>
              </template>
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
    created() {
      this.getMenuList()
      this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
      logout() {
        window.sessionStorage.clear()
        this.$router.push('/login')
      },
      async getMenuList() {
        const {
          data: res
        } = await this.$http.get('menus')
        if (res.meta.status === 200) {
          this.menuList = res.data
        } else {
          this.$message.info(res.meta.msg)
        }
      },
      toggleCollapse() {
        this.isCollapse = !this.isCollapse
      },
      saveNavState(activePath) {
        window.sessionStorage.setItem('activePath', activePath)
        this.activePath = activePath
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
    padding-left: 0;
    color: #fff;
    font-size: 20px;
  }

  .class1 {
    display: flex;
    align-items: center;
  }

  .span {
    margin-left: 15px;
  }

  .el-aside {
    background-color: #333744;
  }

  .el-main {
    background-color: #eaedf1;
  }

  .home_container {
    height: 100%;
  }

  .iconfont {
    margin-right: 10px;
  }

  .toggle_button {
    display: flex;
    background-color: #4a5064;
    font-size: 10px;
    height: 24px;
    color: #fff;
    justify-content: center;
    align-items: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }

</style>
