<template>
    <div style="padding: 20px;">
        <el-card class="box-card">
            <el-form :inline="true" class="demo-form-inline" size="small" :model="searchForm" ref="searchForm">
                <el-form-item label="标题" prop="title">
                    <el-input placeholder="标题" v-model="searchForm.title"></el-input>
                </el-form-item>

                <el-form-item label="公众号" prop="public">
                    <el-select placeholder="请选择公众号" v-model="searchForm.public">
                        <el-option v-for="ele in publicList" :key="ele.id" :label="ele.nick_name" :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="互动类型" prop="interaction">
                    <el-select placeholder="互动类型" v-model="searchForm.interaction">
                        <el-option v-for="ele in templateFansInteractionList" :key="ele.id" :label="ele.text" :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>

            </el-form>

            <el-row type="flex" justify="center">
                <el-button size="small" type="primary" @click="clickSearch">筛选</el-button>
                <el-button size="small" @click="resetSearchForm('searchForm')">重置</el-button>
            </el-row>
        </el-card>

        <el-card class="box-card" style="margin-top: 25px;">
            <div slot="header" class="clearfix">
                <span><i class="el-icon-s-grid"></i> 消息列表</span>

                <el-button-group style="float: right;">
                    <el-button size="small" type="primary" @click="serviceDialogVisible = true">客服消息</el-button>
                    <el-button size="small" type="primary" @click="templateDialogVisible = true">模板消息</el-button>
                </el-button-group>

            </div>
            <div>
                <el-table :data="tableData" border style="width: 100%">
                    <el-table-column prop="title" label="标题"></el-table-column>
                    <el-table-column prop="public_text" label="公众号"></el-table-column>
                    <el-table-column prop="msg_type_text" label="消息类型"></el-table-column>
                    <el-table-column prop="interaction_text" label="粉丝"></el-table-column>
                    <el-table-column prop="count" label="数量"></el-table-column>
                    <el-table-column prop="status_text" label="状态"></el-table-column>
                    <el-table-column label="状态">
                        <template slot-scope="scope">
                            <el-tag v-if="scope.row.status === 1" type="info">{{scope.row.status_text}}</el-tag>
                            <el-tag v-else-if="scope.row.status === 2">{{scope.row.status_text}}</el-tag>
                            <el-tag v-else type="success">{{scope.row.status_text}}</el-tag>
                        </template>

                    </el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-popconfirm title="确定删除吗？" @confirm="confirmDelete(scope.row)">
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


        <el-dialog title="客服消息" :visible.sync="serviceDialogVisible">

            <el-form label-position="left" label-width="80px" :model="serviceForm" ref="serviceForm"
                     :rules="serviceFormRules" size="small">
                <el-form-item label="标题" prop="title" :error="serviceFormError.title">
                    <el-input autocomplete="off" v-model="serviceForm.title"></el-input>
                </el-form-item>
                <el-form-item label="公众号" prop="public" :error="serviceFormError.public">
                    <el-select placeholder="请选择公众号" v-model="serviceForm.public">
                        <el-option v-for="ele in publicList" :key="ele.id" :label="ele.nick_name"
                                   :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="文本" prop="content" :error="serviceFormError.content">
                    <el-input type="textarea" v-model="serviceForm.content"></el-input>
                </el-form-item>

                <el-form-item label="图片" prop="img" :error="serviceFormError.img">
                    <el-upload
                            class="upload-demo"
                            drag
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
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="serviceDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="clickDialogFormSubmit()">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog title="模板消息" :visible.sync="templateDialogVisible">

            <el-form label-position="left" label-width="80px" :model="templateForm" ref="templateForm"
                     :rules="templateFormRules" size="small">

                <el-form-item label="标题" prop="title" :error="templateFormError.title">
                    <el-input autocomplete="off" v-model="templateForm.title"></el-input>
                </el-form-item>

                <el-form-item label="互动类型" prop="public" :error="templateFormError.interaction">
                    <el-select placeholder="互动类型" v-model="templateForm.interaction">
                        <el-option v-for="ele in templateFansInteractionList" :key="ele.id" :label="ele.text"
                                   :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="公众号" prop="public" :error="templateFormError.public">
                    <el-select placeholder="请选择公众号" v-model="templateForm.public" @change="templateChangePublic">
                        <el-option v-for="ele in publicList" :key="ele.id" :label="ele.nick_name"
                                   :value="ele.id"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="消息模板" prop="template_id" :error="templateFormError.template_id">
                    <el-select placeholder="请选择模板" v-model="templateForm.template_id" @change="templateChangeTemplate">
                        <el-option v-for="(ele,tid) in templateDict" :key="tid" :label="ele.title"
                                   :value="tid"></el-option>
                    </el-select>
                </el-form-item>

                <el-row v-if="templateForm.templateItemDict">
                    <el-col :span="12">
                        <el-form-item v-for="(value,name) in templateForm.templateItemDict" :key="name">
                            <el-input autocomplete="off" v-model="templateForm.templateItemDict[name]"
                                      :placeholder="name"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="10" :offset="2">
                        <pre>{{templateItemContent}}</pre>
                    </el-col>
                </el-row>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="templateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="clickTemplateDialogSubmit()">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    import {getToken} from "@/plugins/cookie";

    export default {
        name: "Push",
        data() {
            return {
                // 搜索
                searchForm: {
                    title: "",
                    public: "",
                    interaction: "",
                },
                //表格数据
                tableData: [],

                // 分页配置
                page: {
                    totalCount: 0,
                    perPageSize: 0,
                    currentPage: 1
                },
                dialogVisible: false,

                // 公众号列表
                publicList: [],

                // 客服消息
                serviceDialogVisible: false,
                serviceForm: {
                    title: "",
                    public: "",
                    content: "",
                    img: ""
                },
                serviceFormError: {
                    title: "",
                    public: "",
                    content: "",
                    img: "",
                },
                serviceFormRules: {
                    title: [
                        {required: true, message: '请输入标题', trigger: 'blur'},
                    ],
                    public: [
                        {required: true, message: '请选择公众号', trigger: 'blur'},
                    ],
                },

                // 上传图片
                imageUploadUrl: "http://mtb.pythonav.com/api/base/upload/",
                imageUploadHeaders: {
                    "Authorization": getToken()
                },

                // 模板消息
                templateDialogVisible: false,
                templateForm: {
                    title: "",
                    public: "",
                    template_id: "", // 模板ID iPk5sOIt5X_flOVKn5GrTFpncEYTojx6ddbt8WYoV5s
                    interaction: 1,
                    templateItemDict: ""  // {"result":"", "withdrawMoney":"", ....}
                },
                templateFormError: {
                    title: "",
                    public: "",
                    template_id: "",
                    interaction: ""
                },
                templateFormRules: {
                    title: [
                        {required: true, message: '请输入标题', trigger: 'blur'},
                    ],
                    public: [
                        {required: true, message: '请选择公众号', trigger: 'blur'},
                    ],
                    template_id: [
                        {required: true, message: '请选择模板', trigger: 'blur'},
                    ],
                    interaction: [
                        {required: true, message: '请选择互动类型', trigger: 'blur'},
                    ],
                },
                templateDict: {},
                templateItemContent: "",
                templateFansInteractionList: [
                    {id: 1, text: "48小时互动"},
                    {id: 2, text: "不限"},
                ],

            }
        },
        created() {
            // 加载当前用户已绑定的公众号信息
            this.initPublicList();
            // 加载表格数据
            this.initTableData(1);
        },
        methods: {
            // 加载页面表格数据
            initTableData(page) {
                const loading = this.$loading();

                let searchParams = {page: page};

                for (let key in this.searchForm) {
                    let value = this.searchForm[key];
                    if (value) {
                        searchParams[key] = value;
                    }
                }

                // /msg/message/?page=1 + 搜索条件
                this.axios.get("/msg/message/", {params: searchParams}).then(res => {
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
                this.initTableData(1);
            },
            resetSearchForm(formName) {
                this.$refs[formName].resetFields();
            },

            // 删除
            confirmDelete(row) {
                // `/msg/message/11/`
                this.axios.delete(`/msg/message/${row.id}/`,).then(res => {
                    if (res.data.code === 0) {
                        this.$message.success("删除成功");
                        // 刷新当前页 或 删除数据
                        this.initTableData(this.page.currentPage);
                    } else {
                        this.$message.error(res.data.detail);
                    }
                })
            },

            // 点击分页
            handleChangePage(page) {
                // 当前页
                this.page.currentPage = page;

                // 根据页码加载数据
                this.initTableData(page);
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
                    this.serviceForm.img = response.data.url;
                } else {
                    this.$message.error('上传失败：' + response.error);
                }

            },
            // 客服消息
            clearServiceFormError() {
                for (let key in this.serviceFormError) {
                    this.serviceFormError[key] = ""
                }

            },
            validateServiceFormFailed(errorData) {
                for (let fieldName in errorData) {
                    this.serviceFormError[fieldName] = errorData[fieldName][0];
                }
            },
            clickDialogFormSubmit() {
                // 清空原来的错误
                this.clearServiceFormError();

                // 提交消息
                this.$refs.serviceForm.validate((valid) => {
                    if (!valid) {
                        return false;
                    }

                    this.axios.post("/msg/service/message/", this.serviceForm).then(res => {
                        if (res.data.code === 0) {
                            this.serviceDialogVisible = false;
                            this.$message.success("客服消息添加成功");
                            this.$refs.serviceForm.resetFields();
                            return;
                        }

                        // 1000，字段错误，把相关错误信息现在标签上
                        if (res.data.code === 1000) {
                            // 不好弄，API获取数据，错误显示表单。
                            // detail = { username:['错误',] ,password: [11,22] }
                            this.validateServiceFormFailed(res.data.detail);
                            return;
                        }

                        // 1001，整体错误，整体显示
                        if (res.data.code === 1001) {
                            this.$message.error(res.data.detail);
                        } else {
                            this.$message.error("请求失败");
                        }

                    })
                });

            },

            // 模板消息
            templateChangePublic(publicId) {
                this.templateForm.template_id = "";
                this.templateItemDict = {}

                // /msg/template/?pid=111
                this.axios.get("/msg/template/", {params: {pid: publicId}}).then(res => {
                    if (res.data.code === 0) {
                        this.templateDict = res.data.data;
                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },
            templateChangeTemplate(template_id) {
                this.templateForm.templateItemDict = this.templateDict[template_id].item_dict;
                this.templateItemContent = this.templateDict[template_id].content;
            },
            clickTemplateDialogSubmit() {

                this.axios.post("/msg/template/message/", this.templateForm).then(res => {

                    if (res.data.code === 0) {
                        this.templateDialogVisible = false;
                        this.$message.success("模板息添加成功");
                        this.$refs.templateForm.resetFields();
                        this.templateForm.templateItemDict = "";
                        this.templateItemContent = ""
                        return;
                    }

                    this.$message.error("请求失败");

                })
            }
        }
    }
</script>

<style scoped>

</style>
