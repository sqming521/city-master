<template>
    <div style="padding: 20px;">
        <el-form ref="activeForm" :model="activeForm" label-width="100px" label-position="top" size="small">
            <el-row type="flex" justify="end" style="margin-bottom: 20px;">
                <el-button @click="clickSave()" type="success" size="small"><i class="el-icon-setting"></i> 保存
                </el-button>
            </el-row>

            <el-tabs tab-position="left" v-model="activeName">
                <el-tab-pane label="活动设置" name="tab1">
                    <el-col :span="12">
                        <el-form-item label="活动名称">
                            <el-input v-model="activeForm.name" style="width: 300px;"></el-input>
                        </el-form-item>

                        <el-form-item label="活动时间">
                            <el-date-picker
                                    v-model="activeForm.date_range"
                                    type="datetimerange"
                                    value-format="yyyy-MM-dd HH:mm:ss"
                                    range-separator="至"
                                    start-placeholder="开始日期"
                                    end-placeholder="结束日期">
                            </el-date-picker>
                        </el-form-item>

                        <el-form-item label="拉新保护">
                            <el-switch v-model="activeForm.protect_switch"></el-switch>
                        </el-form-item>

                    </el-col>
                </el-tab-pane>
                <el-tab-pane label="参与公众号" name="tab2">
                    <el-checkbox-group v-model="activeForm.publicList">


                        <el-checkbox v-for="item in totalPublicList" :key="item.id" :label="item.id" border>
                            {{item.nick_name}}（{{item.service_type_info_text}} - {{item.verify_type_info_text}}）
                        </el-checkbox>


                    </el-checkbox-group>
                </el-tab-pane>
                <el-tab-pane label="奖励设置" name="tab3">

                    <el-row :gutter="20" type="flex" justify="space-around">

                        <el-col :span="8" v-for="item in activeForm.awardList" :key="item.level">
                            <el-card class="box-card">
                                <div slot="header" class="clearfix">
                                    <span>{{item.level}}阶任务</span>
                                </div>
                                <div class="text item">
                                    <el-form-item label="人数">
                                        <el-input type="number" v-model="item.count"></el-input>
                                    </el-form-item>
                                    <el-form-item label="奖励">
                                        <el-input v-model="item.goods"></el-input>
                                    </el-form-item>
                                </div>
                            </el-card>
                        </el-col>

                    </el-row>


                </el-tab-pane>
                <el-tab-pane label="海报设置" name="tab4">
                    <el-form-item label="海报关键字">
                        <el-input v-model="activeForm.key" style="width: 200px;"></el-input>
                    </el-form-item>

                    <el-form-item label="活动规则">
                        <el-input type="textarea" v-model="activeForm.rules" style="width: 500px;"></el-input>
                    </el-form-item>

                    <el-form-item label="海报">
                        <el-upload
                                class="upload-demo"
                                drag
                                :show-file-list="false"
                                :limit="1"
                                :multiple="false"
                                :action="imageUploadUrl"
                                :headers="imageUploadHeaders"
                                :on-success="imageUploadSuccess"
                                :before-upload="beforeAvatarUpload">
                            <i class="el-icon-upload"></i>
                            <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                            <div class="el-upload__tip" slot="tip">只能上传jpg文件，且不超过2M</div>
                        </el-upload>
                        <div v-if="previewImgUrl">
                            <img :src="previewImgUrl" style="width: 200px;"/>
                        </div>
                    </el-form-item>

                </el-tab-pane>
            </el-tabs>
        </el-form>
    </div>
</template>

<script>
    import {getToken} from "@/plugins/cookie";

    export default {
        name: "ActivityCreate",
        data() {
            return {
                activeName: "tab1",
                activeForm: {
                    name: '',
                    date_range: "", // [xxx,xxx]
                    protect_switch: true,
                    publicList: [],
                    awardList: [
                        {
                            level: 1,
                            count: 10,
                            goods: ""
                        },
                        {
                            level: 2,
                            count: 20,
                            goods: ""
                        },
                        {
                            level: 3,
                            count: 30,
                            goods: ""
                        },
                    ],
                    key: '',
                    rules: '',
                    img: ""
                },
                totalPublicList: [],
                // 上传图片
                imageUploadUrl: "http://mtb.pythonav.com/api/base/upload/",
                imageUploadHeaders: {
                    "Authorization": getToken()
                },
                previewImgUrl: ""
            }
        },
        created() {
            this.initPublicList();
        },
        methods: {
            //加载我的公众号
            initPublicList() {
                this.axios.get("/base/public/").then(res => {
                    // {code:0,data:[]}
                    if (res.data.code === 0) {
                        this.totalPublicList = res.data.data;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },
            // 上传图片
            beforeAvatarUpload(file) {
                console.log(file.type);
                const isJPG = file.type === 'image/jpeg';
                const isLt2M = file.size / 1024 / 1024 < 2;

                if (!isJPG) {
                    this.$message.error('上传图片只能是 JPG 格式!');
                }
                if (!isLt2M) {
                    this.$message.error('上传图片大小不能超过 2MB!');
                }
                return isJPG && isLt2M;
            },
            imageUploadSuccess(response) {
                if (response.code === 0) {
                    // 1.图片地址+返回时添加
                    // 2.服务器支持访问静态图片
                    // {"code":0,"data":{"url":"/media/uploads/2022/04/01/2_nO3mqV7.jpeg"}}
                    this.activeForm.img = response.data.url;
                    this.previewImgUrl = response.data.abs_url;
                } else {
                    this.$message.error('上传失败：' + response.error);
                }
            },
            // 点击保存
            clickSave() {
                // console.log(this.activeForm);
                //  /api/task/activity/"
                this.axios.post("/task/activity/", this.activeForm).then(res => {
                    if (res.data.code === 0) {
                        // this.totalPublicList = res.data.data;
                        this.$router.push({name: "ActivityList"});
                        this.$message.success("活动添加成功");
                    } else {
                        this.$message.error(res.data.detail);
                    }
                })
            }
        }
    }
</script>

<style scoped>

</style>
