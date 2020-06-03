<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-tickets"></i> 课程详情管理
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class='handle-box'>
                <el-button type="primary" class="handle-del mr10" @click="handleEdit" round>+修改课程参数</el-button>

<!--                &lt;!&ndash; <el-button type="primary" icon="el-icon-delete" class="handle-del mr10" @click="delAllSelection" round>删除所选课程</el-button> &ndash;&gt;-->

                <el-input v-model="query.studentName" placeholder="学生名" class="handle-input mr10"></el-input>

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
                <el-table-column prop="studentId" label="ID" width="55" align="center"></el-table-column>
                <el-table-column prop="studentName" label="学生名"></el-table-column>
                <el-table-column prop="studentExp" label="经验值"></el-table-column>

<!--                <el-table-column label="操作" width="180" align="center">-->
<!--                    <template slot-scope="scope">-->
<!--                        <el-button-->
<!--                                type="text"-->
<!--                                icon="el-icon-plus"-->
<!--                                @click="handleDetail(scope.$index, scope.row)"-->
<!--                        >详情</el-button>-->
<!--                        <el-button-->
<!--                                type="text"-->
<!--                                icon="el-icon-delete"-->
<!--                                class="red"-->
<!--                                @click="handleDelete(scope.$index, scope.row)"-->
<!--                        >删除</el-button>-->
<!--                    </template>-->
<!--                </el-table-column>-->
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
                    studentName: ""
                },
                tableData: [],
                selectTotal: 0,
                multipleSelection: [],
                delList: [],
                delIdList: [],
                editVisible: false,
                addVisible: false,
                form: {
                    studentId: 1,
                    studentName: "",
                    studentEXP: 1
                },
                courseId: 0,
                courseName: "",
                idx: -1,
                id: -1
            };
        },
        created() {
        },
        beforeRouteEnter(to, from, next) {
            next(vm => {
                console.log(vm);
                // 每次进入路由执行
                vm.initRouter();
                vm.getData();
                vm.getDataCount();
            });
        },
        methods: {
            initRouter() {
                this.courseId = parseInt(localStorage.getItem("courseId"));
                this.courseName = localStorage.getItem("courseName");
            },
            // 获取 easy-mock 的模拟数据
            getData() {
                axios
                    .post(
                        "http://localhost:8080/daoyunWeb/courseStudent/getCourseStudentByPage",
                        {
                            page: this.query.page,
                            pageSize: this.query.pageSize,
                            userName: this.query.studentName,
                            courseId: this.courseId
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
                        "http://localhost:8080/daoyunWeb/courseStudent/getCourseStudentCount",
                        { userName: this.query.studentName,
                                courseId: this.courseId
                         },
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
            // 触发搜索按钮
            handleSearch() {
                this.$set(this.query, "pageIndex", 1);
                this.getData();
                this.getDataCount();
            },
            // 多选操作
            handleSelectionChange(val) {
                this.multipleSelection = val;
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
