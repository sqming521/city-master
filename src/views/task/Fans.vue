<template>
    <div style="padding: 20px;">
        <!--搜索区域-->
        <el-card class="box-card">
            <el-form :inline="true" class="demo-form-inline" size="small" :model="searchForm" ref="searchForm">
                <el-form-item label="昵称" prop="name">
                    <el-input placeholder="昵称" v-model="searchForm.name"></el-input>
                </el-form-item>

                <el-form-item label="活动" prop="activity">
                    <el-select placeholder="请选择活动" v-model="searchForm.activity">
                        <el-option label="请选择活动" value=""></el-option>
                        <el-option v-for="ele in activityList" :key="ele.id" :label="ele.name"
                                   :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>


                <el-form-item label="渠道" prop="promo">
                    <el-select placeholder="请选择渠道" v-model="searchForm.promo">
                        <el-option label="请选择渠道" value=""></el-option>
                        <el-option v-for="ele in promoList" :key="ele.id" :label="ele.name" :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item>
                    <el-button size="small" type="primary" @click="clickSearch">筛选</el-button>
                    <el-button size="small" @click="resetSearchForm('searchForm')">重置</el-button>
                </el-form-item>

            </el-form>

        </el-card>

        <el-row style="margin-top: 30px;">

            <el-row justify="end" type="flex" style="float: right;z-index: 1">
                <el-button size="small" type="primary" @click="clickExportExcel">导出</el-button>
                <el-button size="small" type="primary" @click="toBlackList">移入黑名单</el-button>
                <el-button size="small" type="primary" @click="outBlackList">移出黑名单</el-button>
            </el-row>

            <el-tabs v-model="activeName" type="card">
                <el-tab-pane label="参与用户" name="first">

                    <el-table ref="myTable" :data="tableData" style="width: 100%"
                              @selection-change="handleSelectionChange">
                        <el-table-column type="selection" width="55"></el-table-column>
                        <el-table-column prop="nick_name" label="昵称"></el-table-column>
                        <el-table-column prop="open_id" label="OpenID"></el-table-column>
                        <el-table-column label="状态">
                            <template slot-scope="scope">
                                <el-tag v-if="scope.row.looking === 0" type="success">{{scope.row.looking_text}}
                                </el-tag>
                                <el-tag v-else type="danger">{{scope.row.looking_text}}</el-tag>
                            </template>
                        </el-table-column>
                        <el-table-column prop="number" label="数量"></el-table-column>
                        <el-table-column prop="task_progress" label="任务状态"></el-table-column>

                    </el-table>

                    <el-row type="flex" justify="end" style="margin-top: 30px;">
                        <el-pagination
                                :total="page.totalCount"
                                :page-size="page.perPageSize"
                                background
                                layout="prev, pager, next,jumper"
                                @current-change="handlePageChange"
                        >
                        </el-pagination>
                    </el-row>
                </el-tab-pane>


                <el-tab-pane label="黑名单" name="second">
                    <el-table :data="blackTableData" border style="width: 100%" @selection-change="handleBlackSelectionChange">
                        <el-table-column type="selection" width="55"></el-table-column>
                        <el-table-column prop="nick_name" label="昵称"></el-table-column>
                        <el-table-column prop="open_id" label="OpenID"></el-table-column>
                        <el-table-column label="状态">
                            <template slot-scope="scope">
                                <el-tag v-if="scope.row.looking === 0" type="success">{{scope.row.looking_text}}
                                </el-tag>
                                <el-tag v-else type="danger">{{scope.row.looking_text}}</el-tag>
                            </template>
                        </el-table-column>
                        <el-table-column prop="number" label="数量"></el-table-column>
                    </el-table>
                    <el-row type="flex" justify="end" style="margin-top: 30px;">
                        <el-pagination
                                :total="blackPage.totalCount"
                                :page-size="blackPage.perPageSize"
                                background
                                layout="prev, pager, next,jumper"
                                @current-change="handleBlackPageChange"
                        >
                        </el-pagination>
                    </el-row>
                </el-tab-pane>
            </el-tabs>

        </el-row>

    </div>
</template>

<script>
    import {getToken} from "@/plugins/cookie"

    export default {
        name: "Fans",
        data() {
            return {
                activityList: [],
                promoList: [],
                //tab切换
                activeName: "first",
                // 搜索
                searchForm: {
                    name: "",
                    activity: "",
                    promo: "",
                },

                // 白名单
                tableData: [],
                page: {
                    totalCount: 0,
                    perPageSize: 20,
                    currentPage: 1,
                },
                checkList: [],


                // 黑名单
                blackTableData: [],
                blackPage: {
                    totalCount: 0,
                    perPageSize: 20,
                    currentPage: 1,
                },
                blackCheckList: []
            }
        },
        mounted() {
            // this.$refs.myTable.toggleRowSelection(this.tableData[3]);
        },
        created() {
            // 加载活动
            this.initActivity();

            // 加载渠道
            this.initPromo();
        },
        methods: {
            // 加载活动
            initActivity() {
                this.axios.get("/task/activity/",).then(res => {
                    if (res.data.code === 0) {
                        this.activityList = res.data.data;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },

            // 加载渠道
            initPromo() {
                this.axios.get("/task/total/promo/",).then(res => {
                    if (res.data.code === 0) {
                        this.promoList = res.data.data;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },

            // 构造条件
            createCondition() {
                let status = false;
                let searchParams = {};
                for (let key in this.searchForm) {
                    let value = this.searchForm[key];
                    if (value) {
                        status = true;
                        searchParams[key] = value;
                    }
                }
                if (!status) {
                    this.$message.error("请输入搜索条件");
                    return false;
                }
                return searchParams;
            },

            // 获取表格数据
            getTableData(page) {
                this.page.currentPage = page;
                // {name:"root",page:1,black:0}
                let searchParams = this.createCondition();
                if (!searchParams) {
                    return;
                }

                searchParams.page = page;
                searchParams.black = 0;

                const loading = this.$loading();
                this.axios.get("/task/fans/", {params: searchParams}).then(res => {
                    // {code:0,data:{ data:数据,total:总数据条数,page_size:每页几条数据 }}
                    loading.close();
                    if (res.data.code === 0) {
                        this.tableData = res.data.data.data;
                        this.page.totalCount = res.data.data.total;
                        this.page.perPageSize = res.data.data.page_size;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },

            // 获取黑名单数据
            getBlackTableData(page) {
                // {name:"root",page:1,black:1}
                this.blackPage.currentPage = page;
                let searchParams = this.createCondition();
                if (!searchParams) {
                    return;
                }
                searchParams.page = page;
                searchParams.black = 1;

                const loading = this.$loading();
                this.axios.get("/task/fans/", {params: searchParams}).then(res => {
                    // {code:0,data:{ data:数据,total:总数据条数,page_size:每页几条数据 }}
                    loading.close();
                    if (res.data.code === 0) {
                        this.blackTableData = res.data.data.data;
                        this.blackPage.totalCount = res.data.data.total;
                        this.blackPage.perPageSize = res.data.data.page_size;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },


            clickSearch() {
                this.getTableData(1);
                this.getBlackTableData(1);
            },

            resetSearchForm(formName) {
                this.$refs[formName].resetFields();
            },
            // 分页
            handlePageChange(page) {
                this.getTableData(page);
            },
            // 黑名单分页
            handleBlackPageChange(page) {
                this.getBlackTableData(page);
            },


            handleSelectionChange(valueList) {
                this.checkList = valueList;
            },
            handleBlackSelectionChange(valueList) {
                this.blackCheckList = valueList;
            },
            toBlackList() {
                console.log(this.checkList); // [ {},{},{}]

                let idList = [];
                for (let idx in this.checkList) {
                    let item = this.checkList[idx];
                    idList.push(item.id);
                }
                if (idList.length < 1) {
                    return
                }


                this.axios.post("/task/to/black/", {id_list: idList}).then(res => {
                    if (res.data.code === 0) {
                        this.getTableData(this.page.currentPage);
                        this.getBlackTableData(this.blackPage.currentPage);
                    } else {
                        this.$message.error("请求失败");
                    }
                })

            },

            outBlackList() {
                console.log(this.blackCheckList);
                let idList = [];
                for (let idx in this.blackCheckList) {
                    let item = this.blackCheckList[idx];
                    idList.push(item.id);
                }
                if (idList.length < 1) {
                    return
                }

                this.axios.post("/task/out/black/", {id_list: idList}).then(res => {
                    if (res.data.code === 0) {
                        this.getTableData(this.page.currentPage);
                        this.getBlackTableData(this.blackPage.currentPage);
                    } else {
                        this.$message.error("请求失败");
                    }
                })

            },

            // 导出Excel
            clickExportExcel() {
                // {... token:"jwttoken",name:"xx",age:xxx}
                let paramsDict = this.createCondition();
                if (!paramsDict) {
                    return;
                }
                paramsDict.token = getToken();

                let paramString = "";
                for (let key in paramsDict) {
                    let value = paramsDict[key];
                    paramString += `${key}=${value}&`;
                }
                // token=jwttoken...&name=xx&age=xx&
                let url = "http://mtb.pythonav.com/api/task/export?" + paramString.slice(0, -1);
                console.log(url);
                window.open(url, "_blank").location;
            },
        }
    }
</script>

<style scoped>

</style>
