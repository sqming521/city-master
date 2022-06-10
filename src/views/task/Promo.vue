<template>
    <div style="padding: 20px;">
        <el-card class="box-card">
            <el-form :inline="true" class="demo-form-inline" size="small" :model="searchForm" ref="searchForm">
                <el-form-item label="名称" prop="name">
                    <el-input placeholder="名称" v-model="searchForm.name"></el-input>
                </el-form-item>

                <el-form-item label="公众号" prop="public">
                    <el-select placeholder="公众号" v-model="searchForm.public">
                        <el-option v-for="ele in publicList" :key="ele.id" :label="ele.nick_name"
                                   :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item>
                    <el-button size="small" type="primary" @click="clickSearch">筛选</el-button>
                    <el-button size="small" @click="resetSearchForm('searchForm')">重置</el-button>
                </el-form-item>

            </el-form>

        </el-card>

        <el-card class="box-card" style="margin-top: 25px;">
            <div slot="header" class="clearfix">
                <span><i class="el-icon-s-grid"></i> 消息列表</span>
                <el-button style="float: right;" type="primary" size="small" @click="clickAddDrawer">
                    <i class="el-icon-circle-plus-outline"></i> 新建推广码
                </el-button>
            </div>
            <div>
                <el-table :data="tableData" border style="width: 100%">
                    <el-table-column prop="name" label="渠道"></el-table-column>
                    <el-table-column prop="public_text" label="公众号"></el-table-column>
                    <el-table-column label="二维码">
                        <template slot-scope="scope">
                            <el-popover
                                    placement="right"
                                    trigger="hover">
                                <div>
                                    <el-image
                                            style="width: 140px; height: 140px"
                                            :src="scope.row.qr"
                                            fit="fit"></el-image>
                                </div>
                                <el-image slot="reference" style="width: 40px; height: 40px" :src="scope.row.qr"
                                          fit="fit"></el-image>
                            </el-popover>

                        </template>

                    </el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">

                            <el-button @click="handleEditClick(scope.row)" type="text" size="small">编辑</el-button>


                            <el-popconfirm style="display: inline-block;margin-left: 10px;" title="这是一段内容确定删除吗？"
                                           @confirm="confirmDelete(scope.row)">
                                <el-button slot="reference" type="text" size="small">删除</el-button>
                            </el-popconfirm>

                        </template>
                    </el-table-column>
                </el-table>
            </div>
            <el-row type="flex" justify="end" style="margin-top: 30px;">
                <el-pagination
                        :total="page.totalCount"
                        :page-size="page.perPageSize"
                        background
                        layout="prev, pager, next,jumper"
                        @current-change="handleChangePage">
                </el-pagination>
            </el-row>
        </el-card>


        <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
            <span>这是一段信息</span>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
              </span>
        </el-dialog>


        <el-drawer title="新建推广码" :visible.sync="drawerVisible" :before-close="handleDrawerClose" direction="ltr">
            <div style="padding: 25px;">
                <el-form label-position="left" label-width="80px" :model="addForm" ref="addForm" size="small">
                    <el-form-item label="渠道名称" prop="name">
                        <el-input autocomplete="off" v-model="addForm.name"></el-input>
                    </el-form-item>
                    <el-form-item label="公众号" prop="public">
                        <el-select placeholder="请选择公众号" v-model="addForm.public">
                            <el-option v-for="ele in publicList" :key="ele.id" :label="ele.nick_name"
                                       :value="ele.id"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button @click="dialogFormVisible = false">取 消</el-button>
                        <el-button type="primary" @click="clickSubmitAdd()">确 定</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-drawer>

    </div>
</template>

<script>
    export default {
        name: "Promo",
        data() {
            return {
                drawerVisible: false,
                publicList: [],
                editId: "",
                addForm: {
                    name: "",
                    public: ""
                },
                searchForm: {
                    name: "",
                    public: ""
                },
                categoryOptions: [],
                tableData: [],
                page: {
                    totalCount: 10000,
                    perPageSize: 20,
                    currentPage: 1
                },
                dialogFormVisible: false,

                dialogVisible: false
            }
        },
        created() {

            this.initPublicList();

            this.initTableData(1);
        },
        methods: {
            //加载我的公众号
            initPublicList() {
                this.axios.get("/base/public/").then(res => {
                    // {code:0,data:[]}
                    if (res.data.code === 0) {
                        this.publicList = res.data.data;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },

            // 点击新建渠道
            clickSubmitAdd() {
                if (this.editId) {
                    this.axios.put(`/task/promo/${this.editId}/`, this.addForm).then(res => {
                        if (res.data.code === 0) {
                            this.$message.success("编辑成功");
                            // 表单清空
                            // 重新加载页面
                            this.initTableData(this.page.currentPage);
                            // 关闭抽屉
                            this.drawerVisible = false;

                        } else {
                            this.$message.error("请求失败");
                        }
                    })
                } else {
                    this.axios.post("/task/promo/", this.addForm).then(res => {
                        if (res.data.code === 0) {
                            this.$message.success("添加成功");
                            // 表单清空
                            // 重新加载页面
                            this.initTableData(1);
                            // 关闭抽屉
                            this.drawerVisible = false;

                        } else {
                            this.$message.error("请求失败");
                        }
                    })
                }
            },

            // 加载页面表格数据
            initTableData(page) {
                this.page.currentPage = page;
                const loading = this.$loading();

                let searchParams = {page: page};

                for (let key in this.searchForm) {
                    let value = this.searchForm[key];
                    if (value) {
                        searchParams[key] = value;
                    }
                }

                // /msg/promo/?page=1 + 搜索条件
                this.axios.get("/task/promo/", {params: searchParams}).then(res => {
                    // {code:0,data:{ data:数据,total:总数据条数,page_size:每页几条数据 }}
                    loading.close();
                    if (res.data.code === 0) {
                        // console.log(res.data.data.data);
                        this.tableData = res.data.data.data;

                        this.page.totalCount = res.data.data.total;
                        this.page.perPageSize = res.data.data.page_size;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },

            clickSearch() {
                this.initTableData(1);
            },
            resetSearchForm(formName) {
                this.$refs[formName].resetFields();
            },

            // 删除
            confirmDelete(row) {
                this.axios.delete(`/task/promo/${row.id}`).then(res => {
                    if (res.data.code === 0) {
                        //this.dataList = res.data.data;
                        this.initTableData(this.page.currentPage);
                    } else {
                        this.$message.error("删除失败");
                    }
                })
            },

            // 点击新建
            clickAddDrawer() {
                this.editId = "";
                this.drawerVisible = true;
            },
            // 关闭抽屉
            handleDrawerClose(done) {
                this.editId = "";
                for (let key in this.addForm) {
                    this.addForm[key] = "";
                }
                done(); // 关闭
            },

            // 点击编辑
            handleEditClick(row) {
                // 赋值
                this.editId = row.id;
                this.addForm.name = row.name;
                this.addForm.public = row.public;
                // 显示抽屉
                this.drawerVisible = true;
            },

            // 点击分页
            handleChangePage(page) {
                // 当前页
                this.page.currentPage = page;

                // 根据页码加载数据
                this.initTableData(page);
            },
        }
    }
</script>

<style scoped>

</style>
