<template>
  <div class="header">
    <!-- 折叠按钮 -->
    <div class="collapse-btn" @click="collapseChage">
      <i v-if="!collapse" class="el-icon-s-fold"></i>
      <i v-else class="el-icon-s-unfold"></i>
    </div>
    <div class="logo">到云后台管理系统</div>
    <div class="header-right">
      <div class="header-user-con">
        <!-- 全屏显示 -->
        <div class="btn-fullscreen" @click="handleFullScreen">
          <el-tooltip effect="dark" :content="fullscreen?`取消全屏`:`全屏`" placement="bottom">
            <i class="el-icon-rank"></i>
          </el-tooltip>
        </div>
        <!-- 消息中心 -->
        <!-- <div class="btn-bell">
          <el-tooltip effect="dark" :content="message?`有${message}条未读消息`:`消息中心`" placement="bottom">
            <router-link to="/tabs">
              <i class="el-icon-bell"></i>
            </router-link>
          </el-tooltip>
          <span class="btn-bell-badge" v-if="message"></span>
        </div> -->
        <!-- 用户头像 -->
        <div class="user-avator">
          <img src="../../assets/img/img.jpg" />
        </div>
        <!-- 用户名下拉菜单 -->
        <el-dropdown class="user-name" trigger="click" @command="handleCommand">
          <span class="el-dropdown-link">
            {{username}}
            <i class="el-icon-caret-bottom"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item divided command="editpassword">修改密码</el-dropdown-item>
            <el-dropdown-item divided command="loginout">退出登录</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
    </div>
    <el-dialog title="修改密码" :visible.sync="editVisible" width="30%">
      <el-form :model="form" :rules="rules" ref="form" label-width="150px">
        <!-- 之前model与rules写的前后顺序不同导致无法触发，不知道是不是elementui的bug -->
        <el-form-item label="请输入旧密码" prop="oldPassword">
          <el-input type="password" v-model="form.oldPassword" placeholder="输入旧密码"></el-input>
        </el-form-item>
        <el-form-item label="输入你的新密码" prop="newPassword">
          <el-input type="password" v-model="form.newPassword" placeholder="输入新密码"></el-input>
        </el-form-item>
        <el-form-item label="再次确认" prop="newPasswordCheck">
          <el-input type="password" v-model="form.newPasswordCheck" placeholder="再次输入新密码，两次要一致"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeEdit">取 消</el-button>
        <el-button type="primary" @click="saveEdit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import bus from "../common/bus";
import axios from "axios";
export default {
  data() {
    var validatePassOld = (rule, value, callback) => {
      //验证旧密码输入是否为空
      if (value === "") {
        callback(new Error("请输入旧密码"));
      } else {
        callback();
      }
    };
    var validatePass = (rule, value, callback) => {
      //验证新密码输入是否为空，以及再次输入是否为空
      if (value === "") {
        callback(new Error("请输入新密码"));
      } else {
        if (this.form.newPasswordCheck !== "") {
          this.$refs.form.validateField("newPasswordCheck");
        }
        callback();
      }
    };
    var validatePass2 = (rule, value, callback) => {
      //验证两次密码是否一致
      if (value === "") {
        callback(new Error("请再次输入新密码"));
      } else if (value !== this.form.newPassword) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };
    return {
      collapse: false,
      fullscreen: false,
      name: "czl",
      message: 2,
      editVisible: false,
      form: {
        oldPassword: "",
        newPassword: "",
        newPasswordCheck: ""
      },
      rules: {
        oldPassword: [{ validator: validatePassOld, trigger: "blur" }],
        newPassword: [{ validator: validatePass, trigger: "blur" }],
        newPasswordCheck: [{ validator: validatePass2, trigger: "blur" }]
      }
    };
  },
  computed: {
    username() {
      let username = localStorage.getItem("ms_userName");
      return username ? username : this.name;
    }
  },
  methods: {
    // 用户名下拉菜单选择事件
    handleCommand(command) {
      if (command == "loginout") {
        localStorage.removeItem("ms_userName");
        localStorage.removeItem("token");
        this.$router.push("/login");
      } else if (command == "editpassword") {
        this.editVisible = true;
      }
    },
    saveEdit() {
      this.$refs.form.validate(valid => {
        //在validator中每一个可能性下都要callback()，不然valid值会获取不到，出现问题
        if (valid) {
          axios
            .post(
              "http://localhost:8080/daoyunWeb/Login/changePassword",
              {
                password: this.form.oldPassword,
                newPassword: this.form.newPassword,
                userName: localStorage.getItem("ms_userName")
              },
              { headers: { "Content-Type": "application/json" } }
            )
            .then(
              res => {
                console.log(res);
                if (res.status == 200) {
                  if (res.data.code == 0) {
                    alert(res.data.msg+"，即将返回登录页，请重新登录！");
                    this.resetForm();
                    this.editVisible = false;
                    localStorage.removeItem("ms_userName");
                    localStorage.removeItem("token");
                    this.$router.push("/login");
                  } else if (res.data.code == -2) {
                    this.$router.push("/login");
                    this.$message.error(res.data.msg);
                  } else {
                    this.$message.error(res.data.msg);
                  }
                }
              },
              error => {
                console.log(error);
              }
            );
        } else {
          this.$message.error("无效的信息输入");
          return false;
        }
      });
    },
    closeEdit() {
      this.resetForm();
      this.editVisible = false;
    },
    resetForm() {
      this.$refs.form.resetFields();
    },
    // 侧边栏折叠
    collapseChage() {
      this.collapse = !this.collapse;
      bus.$emit("collapse", this.collapse);
    },
    // 全屏事件
    handleFullScreen() {
      let element = document.documentElement;
      if (this.fullscreen) {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        } else if (document.webkitCancelFullScreen) {
          document.webkitCancelFullScreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.msExitFullscreen) {
          document.msExitFullscreen();
        }
      } else {
        if (element.requestFullscreen) {
          element.requestFullscreen();
        } else if (element.webkitRequestFullScreen) {
          element.webkitRequestFullScreen();
        } else if (element.mozRequestFullScreen) {
          element.mozRequestFullScreen();
        } else if (element.msRequestFullscreen) {
          // IE11
          element.msRequestFullscreen();
        }
      }
      this.fullscreen = !this.fullscreen;
    }
  },
  mounted() {
    if (document.body.clientWidth < 1500) {
      this.collapseChage();
    }
  }
};
</script>
<style scoped>
.header {
  position: relative;
  box-sizing: border-box;
  width: 100%;
  height: 70px;
  font-size: 22px;
  color: #fff;
}
.collapse-btn {
  float: left;
  padding: 0 21px;
  cursor: pointer;
  line-height: 70px;
}
.header .logo {
  float: left;
  width: 250px;
  line-height: 70px;
}
.header-right {
  float: right;
  padding-right: 50px;
}
.header-user-con {
  display: flex;
  height: 70px;
  align-items: center;
}
.btn-fullscreen {
  transform: rotate(45deg);
  margin-right: 5px;
  font-size: 24px;
}
.btn-bell,
.btn-fullscreen {
  position: relative;
  width: 30px;
  height: 30px;
  text-align: center;
  border-radius: 15px;
  cursor: pointer;
}
.btn-bell-badge {
  position: absolute;
  right: 0;
  top: -2px;
  width: 8px;
  height: 8px;
  border-radius: 4px;
  background: #f56c6c;
  color: #fff;
}
.btn-bell .el-icon-bell {
  color: #fff;
}
.user-name {
  margin-left: 10px;
}
.user-avator {
  margin-left: 20px;
}
.user-avator img {
  display: block;
  width: 40px;
  height: 40px;
  border-radius: 50%;
}
.el-dropdown-link {
  color: #fff;
  cursor: pointer;
}
.el-dropdown-menu__item {
  text-align: center;
}
</style>
