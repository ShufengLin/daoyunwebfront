<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-tickets"></i> 系统参数页面
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-button
          type="primary"
          icon="el-icon-delete"
          class="handle-del mr10"
          @click="handleAdd"
        >新增参数</el-button>
        <el-button
          type="primary"
          icon="el-icon-delete"
          class="handle-del mr10"
          @click="delAllSelection"
        >批量删除</el-button>
        <el-input v-model="query.paperName" placeholder="参数名称" class="handle-input mr10"></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
      </div>
      <el-table
        :data="tableData"
        border
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55" align="center"></el-table-column>
        <el-table-column prop="paperId" label="参数编号" width="55" align="center"></el-table-column>
        <el-table-column prop="paperName" label="参数名称"></el-table-column>
        <el-table-column prop="paperNum" label="一次签到的经验值"></el-table-column>
        <el-table-column prop="paperDetail" label="可签到的距离"></el-table-column>
        <el-table-column label="操作" width="180" align="center">
          <template slot-scope="scope">
            <el-button
              type="text"
              icon="el-icon-plus"
              @click="handleDetail(scope.$index, scope.row)"
            >详情</el-button>
            <el-button
              type="text"
              icon="el-icon-edit"
              @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button>
            <el-button
              type="text"
              icon="el-icon-delete"
              class="red"
              @click="handleDelete(scope.$index, scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
          background
          layout="total, prev, pager, next"
          :current-page="query.page"
          :page-size="query.pageSize"
          :total="selectTotal"
          @current-change="handlePageChange"
        ></el-pagination>
      </div>
    </div>

    <!-- 编辑弹出框 -->
    <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
      <el-form ref="form" :model="form" label-width="70px">
        <el-form-item label="参数编号">
          <el-input v-model.number="form.paperId" disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="参数名称">
          <el-input v-model="form.paperName"></el-input>
        </el-form-item>
        <el-form-item label="一次签到的经验值">
          <el-input v-model.number="form.paperNum"></el-input>
        </el-form-item>
        <el-form-item label="可签到的距离">
          <el-input v-model="form.paperDetail"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEdit">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 新增弹出框 -->
    <el-dialog title="新增" :visible.sync="addVisible" width="30%">
      <el-form ref="addform" :model="addform" label-width="70px">
        <el-form-item label="字典名称">
          <el-input v-model="addform.paperName"></el-input>
        </el-form-item>
        <el-form-item label="Code">
          <el-input v-model.number="addform.paperNum"></el-input>
        </el-form-item>
        <el-form-item label="字典描述">
          <el-input v-model="addform.paperDetail"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveAdd">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchData } from "../../api/index";
import axios from "axios";
export default {
  name: "basetable",
  data() {
    return {
      query: {
        page: 1,
        pageSize: 5,
        paperName: ""
      },
      tableData: [],
      selectTotal: 0,
      multipleSelection: [],
      delList: [],
      delIdList: [],
      editVisible: false,
      addVisible: false,
      form: {
        paperId: 0,
        paperName: "",
        paperNum: 0,
        paperDetail: ""
      },
      addform: {
        paperName: "",
        paperNum: 0,
        paperDetail: ""
      },
      idx: -1,
      id: -1
    };
  },
  created() {
    this.getData();
    this.getDataCount();
  },
  methods: {
    // 获取 easy-mock 的模拟数据
    getData() {
      //TODO 待加入搜索限定参数
      // axios
      //   .get("http://localhost:9999/daoyunWeb/testExample/getAllPaper")
      //   .then(
      //     res => {
      //       console.log(res);
      //       this.tableData=res.data.data;
      //     },
      //     error => {
      //       console.log(error);
      //     }
      //   );
      axios
        .post(
          "http://localhost:9999/daoyunWeb/testExample/getPaperByPage",
          {
            page: this.query.page,
            pageSize: this.query.pageSize,
            paperName: this.query.paperName
          },
          { headers: { "Content-Type": "application/json" } }
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              if (res.data.code == 0) {
                this.tableData = res.data.data;
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
    },
    getDataCount() {
      //TODO 待加入搜索限定参数
      axios
        .post(
          "http://localhost:9999/daoyunWeb/testExample/getPaperCount",
          { paperName: this.query.paperName },
          { headers: { "Content-Type": "application/json" } }
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              this.selectTotal = res.data.data;
            }
          },
          error => {
            console.log(error);
          }
        );
    },
    updatePaper() {
      axios
        .post(
          "http://localhost:9999/daoyunWeb/testExample/updatePaperJson",
          {
            paperId: this.form.paperId,
            paperName: this.form.paperName,
            paperNum: this.form.paperNum,
            paperDetail: this.form.paperDetail
          },
          { headers: { "Content-Type": "application/json" } }
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              if (res.data.code == 0) {
                this.getData();
                this.getDataCount();
              } else if (res.data.code == -2) {
                this.$router.push({ path: "/login" });
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
    },
    addPaper() {
      axios
        .post(
          "http://localhost:9999/daoyunWeb/testExample/addPaperJson",
          {
            paperName: this.addform.paperName,
            paperNum: this.addform.paperNum,
            paperDetail: this.addform.paperDetail
          },
          { headers: { "Content-Type": "application/json" } }
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              if (res.data.code == 0) {
                this.getData();
                this.getDataCount();
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
    },
    deletePaper() {
      console.log(this.form);
      axios
        .post(
          "http://localhost:9999/daoyunWeb/testExample/deletePaperJson/" +
            this.form.paperId
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              if (res.data.code == 0) {
                this.getData();
                this.getDataCount();
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
    },
    deletePaperBatch() {
      axios
        .post(
          "http://localhost:9999/daoyunWeb/testExample/deletePaperBatchJson",
          this.delIdList,
          { headers: { "Content-Type": "application/json" } }
        )
        .then(
          res => {
            console.log(res);
            if (res.status == 200) {
              if (res.data.code == 0) {
                this.getData();
                this.getDataCount();
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
    },
    // 触发搜索按钮
    handleSearch() {
      this.$set(this.query, "pageIndex", 1);
      this.getData();
      this.getDataCount();
    },
    // 删除操作
    handleDelete(index, row) {
      // 二次确认删除
      this.idx = index;
      this.form = row;
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          this.deletePaper();
          this.$message.success("删除成功");
          //this.tableData.splice(index, 1);
        })
        .catch(() => {});
    },
    // 多选操作
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    delAllSelection() {
      const length = this.multipleSelection.length;
      if (length == 0) {
        this.$message.error("请至少选中一项！");
      } else {
        let str = "";
        this.delList = this.delList.concat(this.multipleSelection);
        for (let i = 0; i < length; i++) {
          str += this.multipleSelection[i].paperName + " ";
          this.delIdList.push(this.multipleSelection[i].paperId);
        }
        this.deletePaperBatch();
        this.$message.error(`删除了${str}`);
        this.multipleSelection = [];
      }
    },
    // 编辑操作
    handleEdit(index, row) {
      this.idx = index;
      this.form = row;
      this.editVisible = true;
    },
    // 保存编辑
    saveEdit() {
      this.editVisible = false;
      this.$message.success(`修改第 ${this.idx + 1} 行成功`);
      this.updatePaper();
      //this.$set(this.tableData, this.idx, this.form);
    },
    // 新增操作
    handleAdd() {
      this.addVisible = true;
    },
    // 保存新增
    saveAdd() {
      this.addPaper();
      this.addVisible = false;
    },
    // 详情操作
    handleDetail(index, row) {
      this.idx = index;
      this.form = row;
      localStorage.setItem("paperId", this.form.paperId);
      localStorage.setItem("paperName", this.form.paperName);
      localStorage.setItem("paperNum", this.form.paperNum);
      localStorage.setItem("paperDetail", this.form.paperDetail);
      this.$router.push({
        path: "/testexpanddetail",
        name: "testexpanddetailpage"
        //params: { paperId: this.form.paperId,paperName: this.form.paperName,paperNum: this.form.paperNum,paperDetail: this.form.paperDetail }
      });
    },
    // 分页导航
    handlePageChange(val) {
      this.$set(this.query, "page", val);
      this.getData();
    }
  }
};
</script>

<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}
.table {
  width: 100%;
  font-size: 14px;
}
.red {
  color: #ff0000;
}
.mr10 {
  margin-right: 10px;
}
.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>
