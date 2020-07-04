<template>
  <div class="login-wrap">
    <div class="ms-login">
      <div class="ms-title">到云后台管理系统</div>
      <el-form :model="param" :rules="rules" ref="login" label-width="0px" class="ms-content">
        <el-form-item prop="userName">
          <el-input v-model="param.userName" placeholder="输入用户名">
            <el-button slot="prepend" icon="el-icon-user"></el-button>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            type="password"
            placeholder="输入密码"
            v-model="param.password"
            @keyup.enter.native="submitForm()"
          >
            <el-button slot="prepend" icon="el-icon-lock"></el-button>
          </el-input>
        </el-form-item>
        <div class="login-btn">
          <el-button type="primary" @click="submitForm()">登录</el-button>
        </div>
        <el-link :underline="false" type="primary" class="forget-link" @click="forgetpassword">忘记密码</el-link>
      </el-form>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data: function() {
    return {
      param: {
        userName: "",
        password: ""
      },
      rules: {
        userName: [
          { required: true, message: "请输入用户名", trigger: "blur" }
        ],
        password: [{ required: true, message: "请输入密码", trigger: "blur" }]
      }
    };
  },
  methods: {
    submitForm() {
      this.$refs.login.validate(valid => {
        if (valid) {
          axios
            .post(
              "http://localhost:8080/daoyunWeb/Login/checkLogin",
              this.param,
              { headers: { "Content-Type": "application/json" } }
            )
            .then(
              res => {
                console.log(res);
                if (res.status == 200) {
                  console.log(res);
                  if (res.data.code == 0) {
                    localStorage.setItem("token", res.data.dataPlus);
                    this.$message.success(res.data.msg);
                    localStorage.setItem("ms_userName", this.param.userName);
                    this.$router.push("/");
                  }
                  else{
                    this.$message.error(res.data.msg);
                  }
                }
              },
              error => {
                console.log(error);
              }
            );
        } else {
          this.$message.error("请输入账号和密码");
          console.log("error submit!!");
          return false;
        }
      });
    },
    forgetpassword() {
      this.$router.push({ path: "/checkphone", query: {} });
    }
  }
};
</script>

<style scoped>
.login-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  background-image: url(../../assets/img/login-bg.jpg);
  background-size: 100%;
}
.ms-title {
  width: 100%;
  line-height: 50px;
  text-align: center;
  font-size: 20px;
  color: #fff;
  border-bottom: 1px solid #ddd;
}
.ms-login {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 350px;
  margin: -190px 0 0 -175px;
  border-radius: 5px;
  background: rgba(255, 255, 255, 0.3);
  overflow: hidden;
}
.ms-content {
  padding: 30px 30px;
}
.login-btn {
  text-align: center;
}
.login-btn button {
  width: 100%;
  height: 36px;
  margin-bottom: 10px;
}
.login-tips {
  font-size: 12px;
  line-height: 30px;
  color: #fff;
}
.forget-link {
  position: absolute;
  right: 30px;
}
</style>