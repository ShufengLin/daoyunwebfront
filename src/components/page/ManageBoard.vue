<template>
  <div class="Change">
    <el-row :gutter="20">
      <el-col :span="20">
                <el-card shadow="hover" class="mgb20" style="height:252px;">
                    <div class="user-info">
                        <img src="../../assets/img/img.jpg" class="user-avator" alt />
                        <div class="user-info-cont">
                            <div class="user-info-name">{{info.userName}}</div>
                            <div>{{roleName}}</div>
                        </div>
                    </div>
                    <div class="user-info-list">
                        联系方式：
                        <span>{{info.phoneNumber}}</span>
                    </div>
                    <div class="user-info-list">
                        学校信息：
                        <span>{{info.school+" "+info.academy+" "+info.major}}</span>
                    </div>
                </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      userId: parseInt(localStorage.getItem("ms_userId")),
      info: {
        userName: "",
        school: "",
        academy: "",
        major: "",
        phoneNumber: ""
      },
      roleName:""
    };
  },
  created() {
    this.roleName = localStorage.getItem("ms_roleName"),
    this.getData();
    console.log(this.roleName);
  },
  methods: {
    getData() {
      axios
        .post(
          "http://localhost:8080/daoyunWeb/Login/getUserInfoById",
          {
            userId: this.userId
          },
          { headers: { "Content-Type": "application/json" } }
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              if (res.data.code == 0) {
                this.info = res.data.data;
                this.$message.success(res.data.msg);
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
    }
  }
};
</script>


<style scoped>
.Change {
  /* border:5px solid green; */
  width: 100%;
  height: 1000px;
}

.info {
  /* border: 5px solid black; */
  float: left;
  width: 100%;
}
.info-time {
  width: 20%;
  float: left;
}
.info-address {
  width: 90%;
  /* border: 2px solid blue; */
}
.user-info-det {
  width: 60%;
  height: 200px;
  margin-left: 30px;
  margin-top: 10px;
  /* border: 5px solid  #000000; */
  font-size: 30px;
}
.user-info-det h4 {
  margin-top: 20px;
  margin-left: 150px;
}
.el-row {
    margin-bottom: 20px;
}
.grid-content {
    display: flex;
    align-items: center;
    height: 100px;
}
.grid-cont-right {
    flex: 1;
    text-align: center;
    font-size: 14px;
    color: #999;
}
.grid-num {
    font-size: 30px;
    font-weight: bold;
}
.grid-con-icon {
    font-size: 50px;
    width: 100px;
    height: 100px;
    text-align: center;
    line-height: 100px;
    color: #fff;
}
.grid-con-1 .grid-con-icon {
    background: rgb(45, 140, 240);
}
.grid-con-1 .grid-num {
    color: rgb(45, 140, 240);
}
.grid-con-2 .grid-con-icon {
    background: rgb(100, 213, 114);
}
.grid-con-2 .grid-num {
    color: rgb(45, 140, 240);
}
.grid-con-3 .grid-con-icon {
    background: rgb(242, 94, 67);
}
.grid-con-3 .grid-num {
    color: rgb(242, 94, 67);
}
.user-info {
    display: flex;
    align-items: center;
    padding-bottom: 20px;
    border-bottom: 2px solid #ccc;
    margin-bottom: 20px;
}
.user-avator {
    width: 120px;
    height: 120px;
    border-radius: 50%;
}
.user-info-cont {
    padding-left: 50px;
    flex: 1;
    font-size: 14px;
    color: #999;
}
.user-info-cont div:first-child {
    font-size: 30px;
    color: #222;
}
.user-info-list {
    font-size: 14px;
    color: #999;
    line-height: 25px;
}
.user-info-list span {
    margin-left: 70px;
}
.mgb20 {
    margin-bottom: 20px;
}
.todo-item {
    font-size: 14px;
}
.todo-item-del {
    text-decoration: line-through;
    color: #999;
}
.schart {
    width: 100%;
    height: 300px;
}
</style>
