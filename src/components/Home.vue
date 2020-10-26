<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="~assets/monkey.png" alt="" />
        黑叶猴管理系统
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409eff" :unique-opened="true" :collapse="isCollapse" :collapse-transition="false" :router="true" :default-active="activePath">
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconObj[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <el-menu-item :index="'/' + item.path" v-for="item in item.children" :key="item.id" @click="saveNavState('/' + item.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ item.authName }}</span>
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
  created() {
    this.getMenuList();
    this.activePath = sessionStorage.getItem("activePath");
  },

  data() {
    return {
      menuList: [],
      iconObj: {
        125: "el-icon-user-solid",
        103: "el-icon-unlock",
        101: "el-icon-shopping-bag-2",
        102: "el-icon-s-order",
        145: "el-icon-data-line",
      },
      isCollapse: false,
      activePath: null,
    };
  },

  methods: {
    logout() {
      sessionStorage.clear();
      this.$router.push("/login");
    },
    async getMenuList() {
      const { data: res } = await this.$axios.get("menus");
      if (res.meta.status != 200) return this.$message.error(res.meta.msg);
      this.menuList = res.data;
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse;
    },
    saveNavState(path) {
      this.activePath = path;
      sessionStorage.setItem("activePath", path);
    },
  },
};
</script>

<style scoped>
.home-container {
  height: 100%;
}

.el-header {
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  background-color: #373d41;
}

.el-header div img {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  vertical-align: middle;
  margin-right: 15px;
}

.el-aside {
  background-color: #333744;
}

.el-aside .el-menu {
  border-right: 0;
}

.el-main {
  background-color: #eaedf1;
}

.toggle-button {
  text-align: center;
  font-size: 10px;
  background-color: #4a5064;
  line-height: 24px;
  color: #fff;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>