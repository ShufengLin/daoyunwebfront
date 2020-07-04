<template>
  <div class="sidebar">
    <el-menu
      class="sidebar-el-menu"
      :default-active="onRoutes"
      :collapse="collapse"
      background-color="#324157"
      text-color="#bfcbd9"
      active-text-color="#20a0ff"
      unique-opened
      router
    >
      <template v-for="item in items">
        <template v-if="item.subs">
          <el-submenu :index="item.index" :key="item.index">
            <template slot="title">
              <i :class="item.icon"></i>
              <span slot="title">{{ item.title }}</span>
            </template>
            <template v-for="subItem in item.subs">
              <el-submenu v-if="subItem.subs" :index="subItem.index" :key="subItem.index">
                <template slot="title">{{ subItem.title }}</template>
                <el-menu-item
                  v-for="(threeItem,i) in subItem.subs"
                  :key="i"
                  :index="threeItem.index"
                >{{ threeItem.title }}</el-menu-item>
              </el-submenu>
              <el-menu-item v-else :index="subItem.index" :key="subItem.index">{{ subItem.title }}</el-menu-item>
            </template>
          </el-submenu>
        </template>
        <template v-else>
          <el-menu-item :index="item.index" :key="item.index">
            <i :class="item.icon"></i>
            <span slot="title">{{ item.title }}</span>
          </el-menu-item>
        </template>
      </template>
    </el-menu>
  </div>
</template>

<script>
import bus from "../common/bus";
import axios from "axios";
export default {
  data() {
    return {
      collapse: false,
      items: [
        {
          icon: "el-icon-house",
          index: "manageboard",
          title: "系统首页"
        },
        {
          icon: "el-icon-tickets",
          index: "role",
          title: "角色管理"
        },
        {
          icon: "el-icon-document",
          index: "student",
          title: "学生管理"
        },
        {
          icon: "el-icon-d-caret",
          index: "teacher",
          title: "教师管理"
        },
        {
          icon: "el-icon-service",
          index: "userRole",
          title: "用户管理"
        },
        {
          icon: "el-icon-time",
          index: "course",
          title: "课程管理"
        },
        {
          icon: "el-icon-news",
          index: "dictionary",
          title: "数据字典管理"
        },
        // {
        //     icon: 'el-icon-tickets',
        //     index: 'test',
        //     title: '测试页面'
        // },
        {
          icon: "el-icon-menu",
          index: "testexpanddetail",
          title: "菜单管理"
        }
        // {
        //     icon: 'el-icon-files',
        //     index: '3',
        //     title: '表单相关',
        //     subs: [
        //         {
        //             index: '500',
        //             title: '500异常页面'
        //         },
        //         {
        //             index: 'customerror',
        //             title: '自定义异常页面'
        //         },
        //         {
        //             index: '3-2',
        //             title: '三级菜单',
        //             subs: [
        //                 {
        //                     index: '403',
        //                     title: '三级菜单1'
        //                 },
        //                 {
        //                     index: '404',
        //                     title: '三级菜单2'
        //                 }
        //             ]
        //         }
        //     ]
        // }
      ]
    };
  },
  computed: {
    onRoutes() {
      return this.$route.path.replace("/", "");
    }
  },
  created() {
    // 通过 Event Bus 进行组件间通信，来折叠侧边栏
    bus.$on("collapse", msg => {
      this.collapse = msg;
      bus.$emit("collapse-content", msg);
    });
    this.getUserRole();
  },
  methods: {
    getUserRole() {
      axios
        .post(
          "http://localhost:8080/daoyunWeb/userRole/getUserRoleByUserId",
          { userId: parseInt(localStorage.getItem("ms_userId")) },
          { headers: { "Content-Type": "application/json" } }
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              console.log(res);
              if (res.data.code == 0) {
                localStorage.setItem("ms_roleId", res.data.data.roleId);
                localStorage.setItem("ms_roleName", res.data.data.roleName);
              } else {
                this.$message.error(res.data.msg);
              }
            }
          },
          error => {
            console.log(error);
          }
        );
    }
  }
};
</script>

<style scoped>
.sidebar {
  display: block;
  position: absolute;
  left: 0;
  top: 70px;
  bottom: 0;
  overflow-y: scroll;
}
.sidebar::-webkit-scrollbar {
  width: 0;
}
.sidebar-el-menu:not(.el-menu--collapse) {
  width: 250px;
}
.sidebar > ul {
  height: 100%;
}
</style>
