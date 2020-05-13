<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> 字典
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
        >新增字典</el-button>
        <el-button
                type="primary"
                icon="el-icon-delete"
                class="handle-del mr10"
                @click="delAllSelection"
        >批量删除</el-button>
        <el-input v-model="query.dicName" placeholder="字典名称" class="handle-input mr10"></el-input>
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
        <el-table-column prop="dicId" label="字典编号" width="55" align="center"></el-table-column>
        <el-table-column prop="dicName" label="字典名称"></el-table-column>
        <el-table-column prop="code" label="Code"></el-table-column>
        <el-table-column prop="dicDescription" label="字典描述"></el-table-column>
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
    <!-- 新增弹出框 -->
    <el-dialog title="新增" :visible.sync="addVisible" width="30%">
      <el-form ref="addform" :model="addform" label-width="70px">
        <el-form-item label="字典名称">
          <el-input v-model="addform.dicName"></el-input>
        </el-form-item>
        <el-form-item label="Code">
          <el-input v-model.number="addform.code"></el-input>
        </el-form-item>
        <el-form-item label="字典描述">
          <el-input v-model="addform.dicDescription"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="addVisible = false">取 消</el-button>
      <el-button type="primary" @click="saveAdd">确 定</el-button>
    </span>
    </el-dialog>
    <!-- 编辑弹出框 -->
      <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
        <el-form ref="form" :model="form" label-width="70px">
          <el-form-item label="字典编号">
            <el-input v-model.number="form.dicId" disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="字典名称">
            <el-input v-model="form.dicName"></el-input>
          </el-form-item>
          <el-form-item label="Code">
            <el-input v-model.number="form.code"></el-input>
          </el-form-item>
          <el-form-item label="字典描述">
            <el-input v-model="form.dicDescription"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEdit">确 定</el-button>
      </span>
      </el-dialog>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name:"basetable",
  data () {
    return {
      query: {
        page: 1,
        pageSize: 5,
        dicName: ""
      },
      tableData: [],
      selectTotal: 0,
      multipleSelection: [],
      delList: [],
      delIdList: [],
      editVisible: false,
      addVisible: false,
      form: {
        dicId: 0,
        dicName: "",
        code: "",
        dicDescription: ""
      },
      addform: {
        dicName: "",
        code: "",
        dicDescription: ""
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
    axios
      .post(
              "http://localhost:8080/daoyunWeb/Dictionary/getDicByPage",
              {
                page: this.query.page,
                pageSize: this.query.pageSize,
                dicName: this.query.dicName
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
                "http://localhost:8080/daoyunWeb/Dictionary/getDicCount",
                { dicName: this.query.dicName },
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
    addDic() {
      axios
        .post(
                "http://localhost:8080/daoyunWeb/Dictionary/addDicJson",
                {
                  dicName: this.addform.dicName,
                  code: this.addform.code,
                  dicDescription: this.addform.dicDescription
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
    updateDic() {
      axios
        .post(
          "http://localhost:8080/daoyunWeb/Dictionary/updateDicJson",
          {
            dicId: this.form.dicId,
            dicName: this.form.dicName,
            code: this.form.code,
            dicDescription: this.form.dicDescription
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
    deleteDic() {
      console.log(this.form)
      axios
        .post(
          "http://localhost:8080/daoyunWeb/Dictionary/deleteDicJson/" +
          this.form.dicId
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
    deleteDicBatch() {
      axios
        .post(
          "http://localhost:8080/daoyunWeb/Dictionary/deleteDicBatchJson",
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
    handleDelete (index,row) {
      // 二次确认删除
      this.idx = index;
      this.form = row;
      this.$confirm('确定要删除吗？', '提示', {
        type: 'warning'
      })
        .then(() => {
          this.deleteDic();
          this.$message.success('删除成功');
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
          str += this.multipleSelection[i].dicName + " ";
          this.delIdList.push(this.multipleSelection[i].dicId);
        }
        this.deleteDicBatch();
        this.$message.error(`删除了${str}`);
        this.multipleSelection = [];
      }
    },
    // 编辑操作
    handleEdit (index, row) {
      this.idx = index;
      this.form = row;
      this.editVisible = true
    },
    // 保存编辑
    saveEdit () {
      this.editVisible = false;
      this.$message.success(`修改第 ${this.idx + 1} 行成功`);
      this.updateDic();
    },
    // 新增操作
    handleAdd() {
      this.addVisible = true;
    },
    saveAdd() {
      this.addDic();
      this.addVisible = false;
    },
    handleDetail(index, row) {
      this.idx = index;
      this.form = row;
      localStorage.setItem("dicId", this.form.dicId);
      localStorage.setItem("dicName", this.form.dicName);
      localStorage.setItem("code", this.form.code);
      localStorage.setItem("dicDescription", this.form.dicDescription);
      this.$router.push({
        path: "/dicdetail",
        name: "dicdetailpage"
        //params: { paperId: this.form.paperId,paperName: this.form.paperName,paperNum: this.form.paperNum,paperDetail: this.form.paperDetail }
      });
    },
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
