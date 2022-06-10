<template>
    <div style="padding: 20px;">
        <!--搜索区域-->
        <el-card class="box-card" shadow="never">
            <el-form :inline="true" class="demo-form-inline" size="small">

                <el-form-item label="活动" prop="category" style="margin-bottom: 0">
                    <el-select placeholder="请选择活动" v-model="activity" @change="changeActivity">

                        <el-option v-for="item in activityList" :key="item.id" :label="item.name" :value="item.id"></el-option>

                    </el-select>
                </el-form-item>

            </el-form>

        </el-card>

        <!--折线图-->
        <el-row style="margin-top: 30px;" :gutter="20">
            <el-col :span="12">

                <el-row type="flex" justify="space-between" :gutter="20" style="flex-wrap: wrap">
                    <el-col :span="12">
                        <el-card shadow="never" class="stat-panel">
                            <div class="row">
                                <el-row type="flex" justify="space-between">
                                    <div>
                                        <div style="font-size: 40px;font-weight: bold">{{today.total}}</div>
                                        <div>今日新增</div>
                                    </div>
                                    <div>
                                        <i class="el-icon-attract" style="font-size: 60px;color: #0c8eff"></i>
                                    </div>
                                </el-row>
                                <el-row type="flex" justify="space-between">
                                    <div>昨日数据：{{yesterday.total}}</div>
                                    <div>累计：{{today.total + yesterday.total}}</div>
                                </el-row>
                            </div>
                        </el-card>
                    </el-col>
                    <el-col :span="12">
                        <el-card shadow="never" class="stat-panel">
                            <div class="row">
                                <el-row type="flex" justify="space-between">
                                    <div>
                                        <div style="font-size: 40px;font-weight: bold">{{today.cancel}}</div>
                                        <div>今日取消</div>
                                    </div>
                                    <div>
                                        <i class="el-icon-attract" style="font-size: 60px;color: #0c8eff"></i>
                                    </div>
                                </el-row>
                                <el-row type="flex" justify="space-between">
                                    <div>昨日数据：{{yesterday.cancel}}</div>
                                    <div>累计：{{today.cancel + yesterday.cancel}}</div>
                                </el-row>
                            </div>
                        </el-card>
                    </el-col>
                    <el-col :span="12" style="margin-top: 8px">
                        <el-card shadow="never" class="stat-panel">
                            <div class="row">
                                <el-row type="flex" justify="space-between">
                                    <div>
                                        <div style="font-size: 40px;font-weight: bold">{{today.valid}}</div>
                                        <div>今日净增</div>
                                    </div>
                                    <div>
                                        <i class="el-icon-attract" style="font-size: 60px;color: #0c8eff"></i>
                                    </div>
                                </el-row>
                                <el-row type="flex" justify="space-between">
                                    <div>昨日数据：{{yesterday.valid}}</div>
                                    <div>累计：{{yesterday.valid + today.valid}}</div>
                                </el-row>
                            </div>
                        </el-card>
                    </el-col>
                    <el-col :span="12" style="margin-top: 8px">
                        <el-card shadow="never" class="stat-panel">
                            <div class="row">
                                <el-row type="flex" justify="space-between">
                                    <div>
                                        <div style="font-size: 40px;font-weight: bold">{{today.join}}</div>
                                        <div>今日参加</div>
                                    </div>
                                    <div>
                                        <i class="el-icon-attract" style="font-size: 60px;color: #0c8eff"></i>
                                    </div>
                                </el-row>
                                <el-row type="flex" justify="space-between">
                                    <div>昨日数据：{{yesterday.join}}</div>
                                    <div>累计：{{today.join + yesterday.join}}</div>
                                </el-row>
                            </div>
                        </el-card>
                    </el-col>
                </el-row>

            </el-col>
            <el-col :span="12">

                <el-card class="box-card" shadow="never">
                    <highcharts :callback="myCallback" :options="opt1"
                                style="height: 330px;min-width: 300px"></highcharts>
                </el-card>

            </el-col>
        </el-row>


        <el-row style="margin-top: 30px;" :gutter="20" type="flex" justify="space-between">
            <el-col :xs="12">
                <el-card class="box-card" shadow="never">
                    <div slot="header">
                        <span>xx统计情况</span>
                    </div>
                    <highcharts :options="opt2" style="height: 330px;min-width: 300px"></highcharts>
                </el-card>
            </el-col>
            <el-col :xs="12">
                <el-card class="box-card" shadow="never">
                    <div slot="header">
                        <span>裂变情况</span>
                    </div>
                    <highcharts :options="opt3" style="height: 330px;min-width: 300px"></highcharts>
                </el-card>
            </el-col>
        </el-row>

    </div>
</template>

<script>
    export default {
        name: "Stat",
        data() {
            return {
                activityList: [],
                activity: "",
                yesterday: {
                    total: 0,
                    valid: 0,
                    cancel: 0,
                    join: 0,
                },
                today: {
                    total: 0,
                    valid: 0,
                    cancel: 0,
                    join: 0,
                },
                opt1: {
                    title: {
                        text: '近7天数据统计'
                    },
                    subtitle: {
                        // text: '数据来源：thesolarfoundation.com'
                    },
                    xAxis: {
                        //categories: ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月']
                        categories: []
                    },
                    yAxis: {
                        title: {
                            text: '数量统计'
                        }
                    },
                    legend: {
                        layout: 'horizontal',
                        align: 'center',
                        verticalAlign: 'bottom'
                    },
                    series: [
                        // {
                        //     name:"总数",
                        //     data:[11,22,33,44,55,66]
                        // },
                        // {
                        //     name:"取关",
                        //     data:[11,22,33,44,55,66]
                        // },
                    ],
                    responsive: {
                        rules: [{
                            condition: {
                                maxWidth: 500
                            }
                        }]
                    }
                },

                opt2: {
                    chart: {
                        plotBackgroundColor: null,
                        plotBorderWidth: null,
                        plotShadow: false,
                        type: 'pie'
                    },
                    title: {
                        text: '2018 年浏览器市场份额'
                    },
                    tooltip: {
                        pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
                    },
                    plotOptions: {
                        pie: {
                            allowPointSelect: true,
                            cursor: 'pointer',
                            dataLabels: {
                                enabled: false
                            },
                            showInLegend: true
                        }
                    },
                    series: [{
                        name: 'Brands',
                        colorByPoint: true,
                        data: [{
                            name: 'Chrome',
                            y: 61.41,
                            sliced: true,
                            selected: true
                        }, {
                            name: 'Internet Explorer',
                            y: 11.84
                        }, {
                            name: 'Firefox',
                            y: 10.85
                        }, {
                            name: 'Edge',
                            y: 4.67
                        }, {
                            name: 'Safari',
                            y: 4.18
                        }, {
                            name: 'Other',
                            y: 7.05
                        }]
                    }]
                },
                opt3: {
                    chart: {
                        type: 'column'
                    },
                    title: {
                        text: '月平均降雨量'
                    },
                    subtitle: {
                        text: '数据来源: WorldClimate.com'
                    },
                    xAxis: {
                        categories: [
                            '一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月'
                        ],
                        crosshair: true
                    },
                    yAxis: {
                        min: 0,
                        title: {
                            text: '降雨量 (mm)'
                        }
                    },
                    tooltip: {
                        // head + 每个 point + footer 拼接成完整的 table
                        headerFormat: '<span style="font-size:10px">{point.key}</span><table>',
                        pointFormat: '<tr><td style="color:{series.color};padding:0">{series.name}: </td>' +
                            '<td style="padding:0"><b>{point.y:.1f} mm</b></td></tr>',
                        footerFormat: '</table>',
                        shared: true,
                        useHTML: true
                    },
                    plotOptions: {
                        column: {
                            borderWidth: 0
                        }
                    },
                    series: [{
                        name: '东京',
                        data: [49.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5, 216.4, 194.1, 95.6, 54.4]
                    }, {
                        name: '纽约',
                        data: [83.6, 78.8, 98.5, 93.4, 106.0, 84.5, 105.0, 104.3, 91.2, 83.5, 106.6, 92.3]
                    }, {
                        name: '伦敦',
                        data: [48.9, 38.8, 39.3, 41.4, 47.0, 48.3, 59.0, 59.6, 52.4, 65.2, 59.3, 51.2]
                    }, {
                        name: '柏林',
                        data: [42.4, 33.2, 34.5, 39.7, 52.6, 75.5, 57.4, 60.4, 47.6, 39.1, 46.8, 51.1]
                    }]
                }

            }
        },
        created() {
            this.initActivity();
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

            // 修改活动
            changeActivity() {
                if (this.activity === "") {
                    return
                }
                // /task/stat/?activity=1
                this.axios.get("/task/stat/", {params: {activity: this.activity}}).then(res => {
                    if (res.data.code === 0) {

                        this.opt1.xAxis.categories = res.data.data.name;
                        this.opt1.series = res.data.data.series;

                        this.today = res.data.data.today;
                        this.yesterday = res.data.data.yesterday;

                    } else {
                        this.$message.error("请求失败");
                    }
                })
            },
            myCallback() {

            }
        }
    }
</script>

<style scoped>
    .stat-panel .row {
        height: 140px;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
    }
</style>
