<!-- index.vue -->
<template>
    <el-container>
        <!--查询表单-->
        <el-header class="formDiv">
            <el-form :inline="true" class="demo-form-inline">
                <el-form-item label="所属区域">
                    <Address
                        :area-code="areaCode"
                        :form-code="form.areaCode"
                        :width="'200px'"
                        :id-edit="idEdit"
                        @getAreaCode="getAreaCode"
                        @keyup.enter.native="onSubmit"
                    />
                </el-form-item>

                <el-form-item label="订餐日期">
                    <el-date-picker
                        @change="handleTimeChange"
                        value-format="yyyy-MM-dd"
                        v-model="date"
                        type="month"
                        placeholder="订餐日期"
                        @keyup.enter.native="onSubmit"
                    />
                </el-form-item>

                <el-form-item>
                    <el-button
                        v-has="{class: '查询'}"
                        type="primary"
                        class="base-btn"
                        @click="onSubmit"
                    >查询</el-button>
                    <el-button
                        v-has="{class: '重置'}"
                        type="primary"
                        class="base-btn"
                        @click="onReset"
                    >重置</el-button>
                    <el-button
                        v-has="{class: '导出'}"
                        type="primary"
                        class="base-btn"
                        @click="exportFun"
                    >导出</el-button>
                </el-form-item>
            </el-form>
        </el-header>
        <!--table板块-->
        <el-main class="tableContainer building-top">
            <el-container>
                <!-- <el-header>
                    <div class="total-nav">
                        <div class="title">月度台账</div>
                    </div>
                </el-header>-->
                <el-main>
                    <el-table
                        height="70vh"
                        :cell-style="{textAlign: 'center'}"
                        border
                        :span-method="arraySpanMethod"
                        :data="tableData"
                        style="width: 100%"
                        :header-cell-style="headerStyle"
                        header-row-class-name="table_title"
                        :row-class-name="tableRowClassName"
                    >
                        <el-table-column :label="`${titleDate}月度台账报表`">
                            <el-table-column prop="myIndex" label="序号" width="50"></el-table-column>
                            <el-table-column prop="name" label="所属居委" width="150"></el-table-column>

                            <el-table-column prop="price" label="套餐选择" width="80"></el-table-column>

                            <el-table-column
                                width="52"
                                v-for="(item,index) in monthDay"
                                :label="`${item}`"
                                :key="index"
                            >
                                <template #default="{row}">
                                    <span>{{row.dayCountList[index]}}</span>
                                </template>
                            </el-table-column>
                            <el-table-column width="80" label="合计" prop="total"></el-table-column>
                        </el-table-column>
                    </el-table>
                </el-main>
            </el-container>
        </el-main>
    </el-container>
</template>

<script>
import Address from "@/components/Address";
import {
    getmonthlyLedgerList,
    exportMonthlyLedgerList
} from "@/api/foodAidConfig/monthlyLedger.js";
import {
    exportTitleConstant,
    exportContentConstant,
    exportGoDownloadConstant,
    exportKnowConstant,
    canExport
} from "@/common/constant"; // 常量池
export default {
    name: "monthlyLedger",
    components: { Address },
    data() {
        return {
            titleDate: "", //表格标题显示日期
            date: "", //选择日期控件绑定值
            exportData: "", // 查询成功后赋值
            total: 0, // 总共多少数量
            // 区域全部ID
            areaCode: [],
            ruleForm: { liveArea: "" },
            form: {
                areaCode: [], // 区域表单ID: ''
                date: ""
            },
            monthDay: "",
            idEdit: false,
            headerStyle: { textAlign: "center" },
            tableData: []
        };
    },

    mounted() {
        this.monthDay = this.getCountDays();
        this.getData();
    },
    methods: {
        // 给每个居委的第一个行添加类名,其他行加不同的类名
        tableRowClassName({row, rowIndex}){
            if (row.flag===1){
                return 'first_row'
            } else  {
                return 'another_row'
            }
        },
        // 日期选择控件的值改变时触发
        handleTimeChange(date) {
            this.form.date = date.slice(0, 7);
        },
        // 导出功能
        async exportFun() {
            await this.onSubmit();

            const confirmTitle =
                "相同筛选条件，10分钟内不可重复提交，确认导出吗？";
            this.$confirm(confirmTitle, "导出残疾人信息", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            })
                .then(() => {
                    exportMonthlyLedgerList(this.form)
                        .then(response => {
                            if (response.code === 0) {
                                this.$alert(
                                    exportContentConstant,
                                    exportTitleConstant,
                                    {
                                        confirmButtonText: exportGoDownloadConstant,
                                        cancelButtonText: exportKnowConstant,
                                        showCancelButton: true,
                                        callback: action => {
                                            if (action === "cancel") return;
                                            this.$router.push(
                                                "/dataManagement/exportManagement"
                                            );
                                        }
                                    }
                                );
                            } else {
                                this.$message.warning(response.msg);
                            }
                        })
                        .catch(error => {
                            this.$message.warning(error.msg);
                        });
                })
                .catch(() => {
                    this.$message({
                        type: "info",
                        message: "已取消操作！"
                    });
                });
        },
        getAreaCode(val) {
            this.areaCode = val;
        },
        // 表单查询按钮
        async onSubmit() {
            this.monthDay = this.getCountDays();
            await this.getData(this.form);
        },
        // 表单重置按钮
        onReset() {
            this.areaCode = [];
            this.date = "";
            this.form = {
                areaCode: [], // 区域表单ID: ''
                date: ""
            };
            this.monthDay = this.getCountDays();
            this.idEdit = !this.idEdit;
            this.getData(this.form);
        },
        // 表格合并规则
        arraySpanMethod({ row, column, rowIndex, columnIndex }) {
            const boundary =
                this.tableData.length -
                this.tableData.filter(item => !item.price).length;
            // console.log(boundary);
            if (rowIndex <= boundary) {
                let data = this.tableData; //拿到当前table中数据

                let cellValue = row[column.property]; //当前位置的值

                let SortArr = ["name", "myIndex"]; //需要合并相同值的字段

                if (cellValue && SortArr.includes(column.property)) {
                    let prevRow = data[rowIndex - 1]; //获取到上一条数据

                    let nextRow = data[rowIndex + 1]; //下一条数据

                    if (prevRow && prevRow[column.property] === cellValue) {
                        //当有上一条数据，并且和当前值相等时,当前框删除，保留上一个框

                        return { rowspan: 0, colspan: 0 };
                    } else {
                        let countRowspan = 1;

                        while (
                            nextRow &&
                            nextRow[column.property] === cellValue
                        ) {
                            //当有下一条数据并且和当前值相等时,获取新的下一条

                            nextRow = data[++countRowspan + rowIndex];
                        }

                        if (countRowspan > 1) {
                            return { rowspan: countRowspan, colspan: 1 };
                        }
                    }
                }
            }

            if (rowIndex >= boundary) {
                if (columnIndex === 1) {
                    return [1, 3];
                }
                if (columnIndex === 0 || columnIndex === 2) {
                    return [0, 0];
                }
            }
        },
        // 获取目标月的天数以及设置表格标题显示的数据

        getCountDays() {
            // 设置表格标题显示的数据
            // 如果没有选择日期,就取当前日期
            if (!this.date) {
                console.log(111111);
                const date = new Date();

                this.titleDate =
                    date.getFullYear() + "年" + (date.getMonth() + 1) + "月";
                console.log(this.titleDate);
            } else {
                this.titleDate =
                    this.date.split("-")[0] +
                    "年" +
                    this.date.split("-")[1] +
                    "月";
            }

            //  获取目标月的天数
            const date = new Date(this.form.date ? this.form.date : null);
            //将当前月份加1，下移到下一个月
            date.setMonth(date.getMonth() + 1);
            //将当前的日期置为0，
            date.setDate(0);
            //再获取天数即取上个月的最后一天的天数
            const days = date.getDate();
            return days;
        },
        // 拿表格数据
        async getData() {
            if (this.areaCode.length) {
                this.form.areaCode = this.areaCode[this.areaCode.length - 1];
            }

            let data;
            try {
                const { data: result } = await getmonthlyLedgerList(this.form);

                data = result;
                // 拿到后端返回的表格数据，进行数据处理
            } catch (error) {
                this.$message.error("获取月度台账失败！");
            }

            // 对data判空，如果查询回来的数据为空就另行处理
            if (!data) {
                return (this.tableData = []);
            }

            // 各套餐金额小计
            data.areaVoList.sort((a, b) => a.packageAmount - b.packageAmount);
            const arr1 = data.areaVoList.map(item => {
                const name = item.packageAmount.split(".")[0] + "元小计";
                const price = "";
                const dayCountList = [];
                let total = "";
                item.dateList.forEach(item => {
                    if (item.datetime !== "total") {
                        dayCountList[+item.datetime - 1] = item.count;
                    } else {
                        total = (item.count ? item.count : 0) + " (元)";
                    }
                });

                return { name, price, dayCountList, total };
            });

            // console.log(arr1);
            // 合计客数

            const dayCountList = [];
            let total = "";
            data.personCountList.forEach(item => {
                if (item.datetime !== "total") {
                    dayCountList[item.datetime - 1] = item.count;
                } else {
                    total = (item.count ? item.count : 0) + " (客)";
                }
            });
            const arr2 = [{ name: "合计客数", price: "", dayCountList, total }];

            // 各居委数据
            let myIndex = 0;
            const arr3 = [];
            data.reportFormVos.forEach(item => {
                myIndex++;
                item.areaList.sort((a, b) => a.packageAmount - b.packageAmount);
                item.areaList.forEach((value,index) => {
                    const name = item.areaName;
                    const price = value.packageAmount;
                    const dayCountList = [];
                    let flag;
                    let total;
                    flag = index===0?1:2
                    value.dateList.forEach(k => {
                        if (k.datetime !== "total") {
                            dayCountList[k.datetime - 1] = k.count;
                        } else {
                            total = (k.count ? k.count : 0) + " (客)";
                        }
                    });
                    arr3.push({ name, price, dayCountList, total, myIndex ,flag});
                });
            });

            // 合计金额

            const dayCountList1 = [];
            let total1 = "";
            data.totalAmountList.forEach(item => {
                if (item.datetime !== "total") {
                    dayCountList1[item.datetime - 1] = item.count;
                } else {
                    total1 = (item.count ? item.count : 0) + " (元)";
                }
            });
            const arr4 = [
                {
                    name: "合计金额",
                    price: "",
                    dayCountList: dayCountList1,
                    total: total1
                }
            ];

            const arr = [...arr3, ...arr1, ...arr2, ...arr4];

            arr.forEach(item => {
                for (let i = 0; i < this.monthDay; i++) {
                    if (!item.dayCountList[i]) {
                        item.dayCountList[i] = 0;
                    }
                }
            });

            this.tableData = arr;
        }
    }
};
</script>

<style  scoped>
@import "~@/styles/form.css";
.total-nav .title {
    font-size: 18px;
    font-weight: 700;
}
</style>

<style lang='scss' scoped>
.building-top {
    ::v-deep {
        .el-table th {
            background-color: #eff6ff !important;
        }

        .cell {
            padding: 0 !important;
        }

        thead {
            tr:nth-child(1) {
                th {
                    border-bottom: 1px solid #707070;
                    border-right: 1px solid #707070;
                }
                .cell {
                    opacity: 1;
                    font-size: 24px;
                    font-family: Source Han Sans CN, Source Han Sans CN-Medium;
                    font-weight: 500;
                    color: #495060;
                }
            }
        }

        //  .el-table td, .el-table th.is-leaf {
        //     border-bottom:  1px solid #707070;
        //     border-right: 1px solid #707070;
        //   }

        .el-table--group,
        .el-table--border {
            border-top: 1px solid #707070;
            border-left: 1px solid #707070;
        }
        .el-table th.is-leaf,
        .el-table td {
            border-bottom: 1px solid #707070;
            border-right: 1px solid #707070;
        }
        .el-table tr td {
            border-bottom: 1px solid #707070;
            border-right: 1px solid #707070;
        }

        .el-table tr:nth-last-child(-n + 2) {
            background-color: #eff6ff !important;
        }

        .el-table tbody tr:nth-child(even):nth-last-child(n + 3).another_row {
            td:nth-child(n + 2) .cell {
                color: #f39b31;
            }
        }
        .el-table tbody tr:nth-child(even):nth-last-child(n + 3).first_row {
            td:nth-child(n + 4) .cell {
                color: #f39b31;
            }
        }

        .el-table tbody tr:nth-last-child(n + 3) {
            td:nth-last-child(1) {
                background-color: #eee;
            }
        }
    }
}
</style>
