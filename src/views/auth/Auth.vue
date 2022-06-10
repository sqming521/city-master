<template>
    <div>

        <el-card class="box-card flex-row-center" shadow="hover">
            <div class="flex-col-center">
                <i @click="toAuthorization()" class="el-icon-circle-plus-outline icon"></i>
                <div class="text">添加公众号</div>
            </div>
        </el-card>

        <el-card v-for="item in dataList" :key="item.id" class="box-card box-item" shadow="hover"
                 :body-style="{width:'100%',padding:'20px'}">
            <div class="item flex-row-center">
                <el-avatar size="large" :src="item.avatar"></el-avatar>
            </div>
            <div class="item flex-row-center">{{item.nick_name}}</div>
            <div class="item flex-row-center">
                <div class="flex-row-between" style="width: 100px;font-size: 12px;">
                    <div style="color: gray">{{item.service_type_info_text}}</div>
                    <div style="color: #0c8eff;">{{item.verify_type_info_text}}</div>
                </div>
            </div>
            <el-divider></el-divider>
            <div class="item small flex-row-between">
                <div><i class="el-icon-position"></i> 任务包</div>
                <div class="date">永久</div>
            </div>
            <div class="item small flex-row-between">
                <div><i class="el-icon-bell"></i> 消息宝</div>
                <div class="date">永久</div>
            </div>
        </el-card>

    </div>
</template>

<script>
    export default {
        name: 'Auth',
        data() {
            return {
                dataList: []
            }
        },
        created: function () {
            this.initDataList();
        },
        methods: {
            toAuthorization() {
                // 发送请求获取 微信二维码URL的页面，跳转过去
                // http://mtb.pythonav.com/api/base/wxurl/
                this.axios.get("/base/wxurl/").then(res => {
                    // {code:0,data:{url:"微信跳转URL"}}
                    if (res.data.code === 0) {
                        window.location.href = res.data.data.url;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },
            initDataList() {
                this.axios.get("/base/public/").then(res => {
                    // {code:0,data:[]}
                    if (res.data.code === 0) {
                        this.dataList = res.data.data;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            }
        }
    }
</script>

<style scoped>
    .box-card {
        width: 240px;
        height: 260px;
        float: left;
        margin: 20px;
    }

    .box-item {
        display: flex;
    }

    .box-item .item {
        padding: 5px 0;
    }

    .box-item .small {
        font-size: 14px;
        padding: 10px 0;
        color: #646464;
    }

    .box-item .date {
        font-size: 13px;
        color: #908e8e;
    }

    .flex-row-center {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
    }

    .flex-row-between {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
    }

    .flex-col-center {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .box-card .icon {
        font-size: 50px;
    }

    .box-card .text {
        font-size: 14px;
        margin-top: 8px;
    }
</style>
