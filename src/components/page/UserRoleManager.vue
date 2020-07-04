<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-tickets"></i> 用户管理
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
                >新增</el-button>
                <el-input v-model="query.userName" placeholder="账号名" class="handle-input mr10"></el-input>
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
                <el-table-column prop="id" label="编号" width="55"  v-if="false" align="center"></el-table-column>
                <el-table-column prop="userId" label="用户编号"  v-if="false"></el-table-column>
                <el-table-column prop="userName" label="账号名"  ></el-table-column>
                <el-table-column prop="roleId" label="角色编号"  v-if="false"></el-table-column>
                <el-table-column prop="roleName" label="角色"  ></el-table-column>
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
                <el-form-item label="用户编号"  v-if="false">
                    <el-input v-model.number="form.id" disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="用户账号"  v-if="false" >
                    <el-input v-model.number="form.userId"></el-input>
                </el-form-item>
                <el-form-item label="账号名">
                    <el-input v-model="form.userName"></el-input>
                </el-form-item>
                <el-form-item label="角色">
                    <el-radio-group v-model="form.roleName"  v-for="item in roleList" :key="item.roleId">
                        <el-radio-button :label="item.roleName"></el-radio-button>
                    </el-radio-group>
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
                <el-form-item label="账号名">
                    <el-input v-model="addForm.userName"></el-input>
                </el-form-item>
                <el-form-item label="角色">
                    <el-radio-group v-model="addForm.roleName"  v-for="item in roleList" :key="item.roleId">
                        <el-radio-button :label="item.roleName"></el-radio-button>
                    </el-radio-group>
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
                roleList:[],
                editVisible: false,
                addVisible: false,
                form: {
                    id:0,
                    userId: 0,
                    userName: "",
                    roleId: 0,
                    roleName: "",
                },
                addForm: {
                    id:0,
                    userId: 0,
                    userName: "",
                    roleId: 0,
                    roleName: "",
                },
                idx: -1,
                id: -1
            };
        },
        created() {
            this.getData();
            this.getDataCount();
            this.getAllRoleData();
        },
        methods: {
            // 获取 easy-mock 的模拟数据
            getData() {
                //TODO 待加入搜索限定参数
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/userRole/getUserRoleByPage",
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
                        "http://121.196.49.85:9999/daoyunWeb/userRole/getUserRoleCount",
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
            getAllRoleData() {
                //TODO 待加入搜索限定参数
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/userRole/getAllRole",
                        {
                            page: this.query.page,
                            pageSize: this.query.pageSize,
                        },
                        { headers: { "Content-Type": "application/json" } }
                    )
                    .then(
                        res => {
                            console.log(res);
                            if (res.status == 200) {
                                if (res.data.code == 0) {
                                    this.roleList = res.data.data;
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
            updateUserRole() {
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/userRole/updateUserRoleJson",
                        {
                            id: this.form.id,
                            userId: this.form.userId,
                            userName: this.form.userName,
                            roleId: this.form.roleId,
                            roleName: this.form.roleName,
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
                                    this.getAllRoleData();
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
            // 增加用户
            addUserRole(){
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/userRole/addUserRoleJson",
                        {
                            userName: this.addForm.userName,
                            roleName: this.addForm.roleName
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
                                    this.getAllRoleData();
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
                this.getAllRoleData();
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
                this.addUserRole();
                this.addVisible = false;
            },
            // 保存编辑
            saveEdit() {
                this.editVisible = false;
                this.$message.success(`修改成功`);
                this.updateUserRole();
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
