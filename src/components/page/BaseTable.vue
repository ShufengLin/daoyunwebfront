<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-tickets"></i> 学生管理
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
                >新增学生</el-button>
                <el-input v-model="query.userName" placeholder="学生姓名" class="handle-input mr10"></el-input>
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
                <el-table-column prop="userId" label="学生编号" width="55" align="center" v-if="false"></el-table-column>
                <el-table-column prop="userName" label="学生姓名"></el-table-column>
                <el-table-column prop="phoneNumber" label="电话号码"></el-table-column>
                <el-table-column prop="password" label="密码"></el-table-column>
                <el-table-column prop="school" label="学校"></el-table-column>
                <el-table-column prop="academy" label="学院"></el-table-column>
                <el-table-column prop="major" label="专业"></el-table-column>
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                                type="text"
                                icon="el-icon-edit"
                                @click="handleEdit(scope.$index, scope.row)"
                        >编辑</el-button>
<!--                        <el-button-->
<!--                                type="text"-->
<!--                                icon="el-icon-delete"-->
<!--                                class="red"-->
<!--                                @click="handleDelete(scope.$index, scope.row)"-->
<!--                        >删除</el-button>-->
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
                <el-form-item label="学生编号" v-if="false">
                    <el-input v-model.number="form.userId" disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="学生姓名">
                    <el-input v-model="form.userName"></el-input>
                </el-form-item>
                <el-form-item label="电话号码">
                    <el-input v-model.number="form.phoneNumber"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input v-model="form.password"></el-input>
                </el-form-item>
                <el-form-item label="学校">
                    <el-input v-model="form.school"></el-input>
                </el-form-item>
                <el-form-item label="学院">
                    <el-input v-model="form.academy"></el-input>
                </el-form-item>
                <el-form-item label="专业">
                    <el-input v-model="form.major"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEdit">确 定</el-button>
      </span>
        </el-dialog>

        <!-- 新增弹出框 -->
        <el-dialog title="新增" :visible.sync="addVisible" width="30%">
            <el-form ref="addForm" :model="addForm" label-width="70px">
                <el-form-item label="学生姓名">
                    <el-input v-model="addForm.userName"></el-input>
                </el-form-item>
                <el-form-item label="电话号码">
                    <el-input v-model.number="addForm.phoneNumber"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="学校">
                    <el-input v-model="addForm.school"></el-input>
                </el-form-item>
                <el-form-item label="学院">
                    <el-input v-model="addForm.academy"></el-input>
                </el-form-item>
                <el-form-item label="专业">
                    <el-input v-model="addForm.major"></el-input>
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
import { fetchData } from '../../api/index';
import axios from "axios";
export default {
    name: 'student',
    data() {
        return {
            query: {
                page: 1,
                pageSize: 5,
                userName: ""
            },
            tableData: [],
            selectTotal: 0,
            multipleSelection: [],
            delList: [],
            delIdList: [],
            editVisible: false,
            addVisible: false,
            form: {
                userId: 0,
                userName: "",
                phoneNumber: 0,
                password: 0,
                school: "",
                academy: "",
                major: ""
            },
            addForm: {
                userId: 0,
                userName: "",
                phoneNumber: 0,
                password: 0,
                school: "",
                academy: "",
                major: ""
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
                    "http://localhost:9999/daoyunWeb/student/getStudentByPage",
                    {
                        page: this.query.page,
                        pageSize: this.query.pageSize,
                        userName: this.query.userName
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
                                this.$router.push('/login');
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
                    "http://localhost:9999/daoyunWeb/student/getStudentCount",
                    { userName: this.query.userName },
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
        updateStudent() {
            axios
                .post(
                    "http://localhost:9999/daoyunWeb/student/updateStudentJson",
                    {
                        userId: this.form.userId,
                        userName: this.form.userName,
                        phoneNumber: this.form.phoneNumber,
                        password: this.form.password,
                        school: this.form.school,
                        academy: this.form.academy,
                        major: this.form.major
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
        // 增加课程
        addStudent(){
            axios
                .post(
                    "http://localhost:9999/daoyunWeb/student/addStudentJson",
                    {
                        userId: this.addForm.userId,
                        userName: this.addForm.userName,
                        phoneNumber: this.addForm.phoneNumber,
                        password: this.addForm.password,
                        school: this.addForm.school,
                        academy: this.addForm.academy,
                        major: this.addForm.major
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
                                this.$router.push('/login');
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
            this.$set(this.query, 'pageIndex', 1);
            this.getData();
            this.getDataCount();
        },
        // 删除操作
        handleDelete(index, row) {
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            })
                .then(() => {
                    this.$message.success('删除成功');
                    this.tableData.splice(index, 1);
                })
                .catch(() => {});
        },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        delAllSelection() {
            const length = this.multipleSelection.length;
            let str = '';
            this.delList = this.delList.concat(this.multipleSelection);
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].name + ' ';
            }
            this.$message.error(`删除了${str}`);
            this.multipleSelection = [];
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
        },
        // 新增操作
        handleAdd() {
            this.addVisible = true;
        },
        // 保存新增
        saveAdd() {
            this.addStudent();
            this.addVisible = false;
        },
        // 保存编辑
        saveEdit() {
            this.editVisible = false;
            this.$message.success(`修改第 ${this.idx + 1} 行成功`);
            this.updateStudent();
        },
        // 分页导航
        handlePageChange(val) {
            this.$set(this.query, 'page', val);
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
