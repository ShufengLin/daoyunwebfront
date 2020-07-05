<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-tickets"></i> 角色管理
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class='handle-box'>
                <el-button type="primary" class="handle-del mr10" @click="handleAdd" round>+新增角色</el-button>

                <!-- <el-button type="primary" icon="el-icon-delete" class="handle-del mr10" @click="delAllSelection" round>删除所选课程</el-button> -->

                <el-input v-model="query.roleName" placeholder="角色名" class="handle-input mr10"></el-input>

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
                <el-table-column prop="roleId" label="ID" width="55" align="center" v-if="false"></el-table-column>
                <el-table-column prop="roleName" label="角色名称"></el-table-column>
                <el-table-column prop="roleDescription" label="角色描述"></el-table-column>

                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                                type="text"
                                icon="el-icon-plus"
                                @click="handleDetail(scope.$index, scope.row)"
                        >权限</el-button>
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

        <!-- 查看弹出框 -->
        <el-dialog title="查看" :visible.sync="detailVisible" width="30%">
            <el-form ref="form" :model="viewForm" label-width="70px">
                <el-form-item label="角色编号" v-if="false">
                    <el-input readonly="true" v-model="viewForm.roleId"></el-input>
                </el-form-item>
                <el-form-item label="角色权限" >
                    <el-checkbox-group v-model="ownPermissionList" disabled>
                        <el-checkbox v-for="permission in ownPermissionList" :label="permission" :key="permission">{{permission}}</el-checkbox>
                    </el-checkbox-group>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button type="primary" @click="detailVisible = false">确 定</el-button>
          </span>
        </el-dialog>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="角色名">
                    <el-input v-model="form.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述">
                    <el-input v-model="form.roleDescription"></el-input>
                </el-form-item>
                <el-form-item label="角色权限">
                    <el-checkbox-group v-model="ownPermissionList">
                        <el-checkbox v-for="permission in allPermissionList" :label="permission.permissionName" :key="permission.permissionName">{{permission.permissionName}}</el-checkbox>
                    </el-checkbox-group>
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
                <el-form-item label="角色名">
                    <el-input v-model="addForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述">
                    <el-input v-model="addForm.roleDescription"></el-input>
                </el-form-item>
                <el-form-item label="角色权限">
                    <el-checkbox-group v-model="addForm.permissionList">
                        <el-checkbox v-for="permission in allPermissionList" :label="permission.permissionId" :key="permission.permissionId">{{permission.permissionName}}</el-checkbox>
                    </el-checkbox-group>
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
        name: 'course',
        data() {
            return {
                query: {
                    page:1,
                    pageSize: 5,
                    roleName: ""
                },
                tableData: [],
                selectTotal: 0,
                multipleSelection: [],
                allPermissionList: [],
                ownPermissionList: [],
                delList: [],
                delIdList: [],
                detailVisible: false,
                editVisible: false,
                addVisible: false,
                viewForm: {
                    roleId: 1,
                    roleName: "",
                    roleDescription: "",
                    permissionList: []
                },
                form: {
                    roleId: 1,
                    roleName: "",
                    roleDescription: "",
                    permissionList: []
                },
                addForm: {
                    roleId: 1,
                    roleName: "",
                    roleDescription: "",
                    permissionList: []
                },
                idx: -1,
                id: -1
            };
        },
        created() {
            this.getData();
            this.getDataCount();
            this.getAllPermissionData();
        },
        methods: {
            // 获取 easy-mock 的模拟数据
            getData() {
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/role/getRoleByPage",
                        {
                            page: this.query.page,
                            pageSize: this.query.pageSize,
                            roleName: this.query.roleName
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
                        "http://121.196.49.85:9999/daoyunWeb/role/getRoleCount",
                        { roleName: this.query.roleName },
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
            getAllPermissionData() {
                //TODO 待加入搜索限定参数
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/role/getAllPermission",
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
                                    this.allPermissionList = res.data.data;
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
            updateRole() {
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/role/updateRole",
                        {
                            roleName: this.addForm.roleName,
                            roleDescription: this.addForm.roleDescription,
                            permissionList: this.addForm.permissionList
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
                                    this.getAllPermissionData();
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
            // 增加角色
            addRole(){
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/role/addRoleJson",
                        {
                            roleName: this.addForm.roleName,
                            roleDescription: this.addForm.roleDescription,
                            permissionList: this.addForm.permissionList
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
                                    this.getAllPermissionData();
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
            //获取角色拥有权限
            getOwnPermission() {
                //TODO 待加入搜索限定参数
                axios
                    .post(
                        "http://121.196.49.85:9999/daoyunWeb/role/getOwnPermissionByRoleId",
                        {
                            roleId: this.form.roleId
                        },
                        { headers: { "Content-Type": "application/json" } }
                    )
                    .then(
                        res => {
                            console.log(res);
                            if (res.status == 200) {
                                if (res.data.code == 0) {
                                    this.ownPermissionList = res.data.data;
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
            // 查看权限操作
            handleDetail(index, row) {
                this.idx = index;
                this.form = row;
                this.getOwnPermission();
                this.detailVisible = true;
            },
            // 触发搜索按钮
            handleSearch() {
                this.$set(this.query, "pageIndex", 1);
                this.getData();
                this.getDataCount();
                this.getAllPermissionData();
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
            // 新增操作
            handleAdd() {
                this.addVisible = true;
            },
            // 保存新增
            saveAdd() {
                this.addRole();
                this.addVisible = false;
            },
            // 编辑操作
            handleEdit(index, row) {
                this.idx = index;
                this.form = row;
                this.getOwnPermission();
                this.form.permissionList = this.ownPermissionList;
                this.editVisible = true;
            },
            // 保存编辑
            saveEdit() {
                this.editVisible = false;
                this.$message.success(`修改权限成功`);
                // this.updateRole();
                //this.$set(this.tableData, this.idx, this.form);
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
