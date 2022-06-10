<template>
    <div style="padding: 20px">
        <router-link :to="{name:'ActivityCreate'}">
            <el-button type="primary" size="small"><i class="el-icon-circle-plus-outline"></i> 新建活动</el-button>
        </router-link>

        <el-row style="margin-top: 25px;">
            <el-form :inline="true" class="demo-form-inline" size="small" :model="searchForm" ref="searchForm">

                <el-form-item label="标题" prop="name">
                    <el-input placeholder="标题" v-model="searchForm.name"></el-input>
                </el-form-item>

                <el-form-item label="公众号" prop="public">
                    <el-select placeholder="请选择公众号" v-model="searchForm.public">
                        <el-option v-for="ele in publicList" :key="ele.id" :label="ele.nick_name"
                                   :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item>
                    <el-button size="small" type="primary" @click="clickSearch()">筛选</el-button>
                    <el-button size="small" @click="resetSearchForm('searchForm')">重置</el-button>
                </el-form-item>

            </el-form>

        </el-row>


        <el-row :gutter="20">
            <el-col style="width: 240px;margin-bottom: 20px;" v-for="item in dataList" :key="item.id">
                <el-card :body-style="{ padding: '0px' }">
                    <div style="height: 160px;overflow: hidden">
                        <img :src="item.img" class="image">
                    </div>
                    <div style="padding: 14px;">
                        <span>{{item.name}}</span>
                        <el-row style="margin-top: 8px;">
                            <el-button @click="deleteActivity(item.id)" icon="el-icon-delete" size="mini" circle></el-button>
                            <el-button icon="el-icon-edit" size="mini" circle></el-button>
                            <el-button icon="el-icon-check" size="mini" circle></el-button>
                            <el-button icon="el-icon-message" size="mini" circle></el-button>
                        </el-row>
                    </div>
                </el-card>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    export default {
        name: "ActivityList",
        data() {
            return {
                searchForm: {
                    name: "",
                    public: ""
                },
                categoryOptions: [
                    {
                        value: '1',
                        label: '黄金糕'
                    }, {
                        value: '2',
                        label: '双皮奶'
                    }, {
                        value: '3',
                        label: '蚵仔煎'
                    }, {
                        value: '4',
                        label: '龙须面'
                    }, {
                        value: '5',
                        label: '北京烤鸭'
                    }],
                currentDate: new Date(),
                dataList: [],
                // 公众号列表
                publicList: [],
            }
        },
        created() {
            // 加载数据
            this.initDataList();
            // 加载当前用户已绑定的公众号信息
            this.initPublicList();
        },
        methods: {
            //加载
            initDataList() {

                let searchParams = {};
                for (let key in this.searchForm) {
                    let value = this.searchForm[key];
                    if (value) {
                        searchParams[key] = value;
                    }
                }

                this.axios.get("/task/activity/", {params: searchParams}).then(res => {
                    if (res.data.code === 0) {
                        this.dataList = res.data.data;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },

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

            // 搜索
            clickSearch() {
                this.initDataList();
            },
            resetSearchForm(formName) {
                this.$refs[formName].resetFields();
            },
            // 删除活动
            deleteActivity(id){
                this.axios.delete(`/task/activity/${id}`).then(res => {
                    if (res.data.code === 0) {
                        //this.dataList = res.data.data;
                        this.initDataList();
                    } else {
                        this.$message.error("删除失败");
                    }
                })
            }

        }

    }
</script>

<style scoped>

    .image {
        width: 100%;
        display: block;
    }

</style>
