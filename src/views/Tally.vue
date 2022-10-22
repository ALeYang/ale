<template>
  <div class="tally">
    <div class="button-wrap">
      <el-button
        type="primary"
        round
        size="small"
        icon="el-icon-plus"
        @click="add"
        >新增</el-button
      >
      <el-button
        type="danger"
        round
        size="small"
        icon="el-icon-delete"
        :disabled="display"
        @click="del"
      >
        删除
      </el-button>
    </div>
    <div class="tally-table">
      <el-table
        :data="tableData"
        style="width: 120%"
        max-height="500"
        tooltip-effect="light"
        @selection-change="handleSelectionChange"
        :default-sort="{ prop: 'date' }"
      >
        <el-table-column
          type="selection"
          width="50"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="date"
          label="日期"
          width="110"
          align="center"
          sortable
          fixed
        >
        </el-table-column>
        <el-table-column prop="stock" label="股票" width="120" align="center">
        </el-table-column>
        <el-table-column prop="fund" label="基金" width="120" align="center">
        </el-table-column>
        <el-table-column
          prop="aliPay"
          label="余额宝"
          width="120"
          align="center"
        >
        </el-table-column>
        <el-table-column prop="weChat" label="微信" width="120" align="center">
        </el-table-column>
        <el-table-column prop="bank" label="银行卡" width="120" align="center">
        </el-table-column>
        <el-table-column
          prop="expense"
          label="待报销"
          width="120"
          align="center"
        >
        </el-table-column>
        <el-table-column prop="note" label="待报销项目" align="center">
        </el-table-column>
        <el-table-column
          prop="combined"
          label="合计"
          width="120"
          align="center"
        >
        </el-table-column>
        <el-table-column prop="tag" label="标签" width="120" align="center">
          <template slot-scope="scope">
            <el-tag type="success" size="small" v-if="scope.row.tag === 1"
              >月底</el-tag
            >
          </template>
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="120" align="center">
          <template slot-scope="scope">
            <el-button
              @click.native.prevent="deleteRow(scope.$index, tableData)"
              type="text"
              size="small"
            >
              修改
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="chart-btn">
      <el-switch
        v-model="isLastDay"
        active-text="只看月底"
        inactive-text="全部查看"
      >
      </el-switch>
    </div>
    <div class="tally-chart">
      <div id="overview"></div>
    </div>

    <el-dialog
      :title="title"
      :visible.sync="addOpen"
      width="600px"
      :before-close="reset"
      :close-on-click-modal="false"
    >
      <el-form
        inline
        label-width="auto"
        :rules="addRules"
        ref="addForm"
        :model="form"
      >
        <el-row>
          <el-form-item label="日期" prop="date">
            <el-date-picker
              v-model="form.date"
              style="width: 230%"
              size="small"
              align="right"
              type="date"
              placeholder="选择日期"
              :picker-options="pickerOptions"
            >
            </el-date-picker>
          </el-form-item>
        </el-row>
        <el-form-item label="股票" prop="stock">
          <el-input-number
            placeholder="股票"
            v-model="form.stock"
            size="small"
            style="width: 110%"
            :controls="false"
            :precision="2"
            :min="-9999999"
            :max="9999999"
          >
          </el-input-number>
        </el-form-item>
        <el-form-item label="基金" prop="fund">
          <el-input-number
            placeholder="基金"
            v-model="form.fund"
            size="small"
            style="width: 110%"
            :controls="false"
            :precision="2"
            :min="-9999999"
            :max="9999999"
          >
          </el-input-number>
        </el-form-item>
        <el-form-item label="余额宝" prop="aliPay">
          <el-input-number
            placeholder="余额宝"
            v-model="form.aliPay"
            size="small"
            style="width: 110%"
            :controls="false"
            :precision="2"
            :min="-9999999"
            :max="9999999"
          >
          </el-input-number>
        </el-form-item>
        <el-form-item label="微信" prop="weChat">
          <el-input-number
            placeholder="微信"
            v-model="form.weChat"
            size="small"
            style="width: 110%"
            :controls="false"
            :precision="2"
            :min="-9999999"
            :max="9999999"
          >
          </el-input-number>
        </el-form-item>
        <el-form-item label="银行卡" prop="bank">
          <el-input-number
            placeholder="银行卡"
            v-model="form.bank"
            size="small"
            style="width: 110%"
            :controls="false"
            :precision="2"
            :min="-9999999"
            :max="9999999"
          >
          </el-input-number>
        </el-form-item>
        <el-form-item label="待报销" prop="expense">
          <el-input-number
            placeholder="待报销"
            v-model="form.expense"
            size="small"
            style="width: 110%"
            :controls="false"
            :precision="2"
            :min="-9999999"
            :max="9999999"
          >
          </el-input-number>
        </el-form-item>
        <el-form-item label="待报销项目" prop="note">
          <el-input
            type="textarea"
            placeholder="待报销项目"
            v-model="form.note"
            style="width: 220%"
            size="small"
            clearable
          >
          </el-input>
        </el-form-item>
      </el-form>
      <div class="button" style="text-align: right">
        <el-button type="primary" round size="small" @click="submit"
          >确认</el-button
        >
        <el-button type="info" round size="small" @click="reset"
          >取消</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
import * as echarts from "echarts";

export default {
  data() {
    return {
      // dialog控制
      addOpen: false,

      // 删除按钮展示
      display: true,

      // 选中数据
      ids: [],

      // 数据
      tableData: [
        {
          id: 1,
          date: "2016-05-02",
          stock: "11111.11",
          fund: "2222",
          aliPay: "3333",
          weChat: "4444",
          bank: "5555",
          expense: "6666",
          note: "备注",
          combined: "7777",
          tag: 0,
        },
        {
          id: 2,
          date: "2016-05-01",
          stock: "11111.11",
          fund: "2222",
          aliPay: "3333",
          weChat: "4444",
          bank: "5555",
          expense: "6666",
          note: "备注",
          combined: "7777",
          tag: 1,
        },
      ],
      pickerOptions: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
        shortcuts: [
          {
            text: "今天",
            onClick(picker) {
              picker.$emit("pick", new Date());
            },
          },
          {
            text: "昨天",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1200 * 24);
              picker.$emit("pick", date);
            },
          },
          {
            text: "一周前",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1200 * 24 * 7);
              picker.$emit("pick", date);
            },
          },
        ],
      },

      // chartBtn
      isLastDay: false,

      // dialog表单
      title: "",
      form: {
        date: "",
      },
      addRules: {
        date: [
          {
            type: "date",
            required: true,
            message: "请选择日期",
            trigger: "blur",
          },
        ],
        stock: [{ required: true, message: "请填入股票", trigger: "blur" }],
        fund: [{ required: true, message: "请填入基金", trigger: "blur" }],
        aliPay: [{ required: true, message: "请填入余额宝", trigger: "blur" }],
        weChat: [{ required: true, message: "请填入微信", trigger: "blur" }],
        bank: [{ required: true, message: "请填入银行卡", trigger: "blur" }],
        expense: [{ required: true, message: "请填入待报销", trigger: "blur" }],
        note: [
          { required: false, message: "请填入待报销项目", trigger: "blur" },
        ],
      },
    };
  },
  mounted() {
    this.getTable();
    this.echartInitialize();
  },
  methods: {
    // 初始化table
    getTable() {
      this.axios
        .post("/assest/add", {
          name: "tom",
        })
        .then((res) => {
          console.log(res);
        });
      this.axios
        .delete("/assest/del", {
          params: {
            name: "tom",
          },
        })
        .then((res) => {
          console.log(res);
        });
    },

    // 复选框
    handleSelectionChange(val) {
      this.ids = [];
      this.display = true;
      if (val.length >= 1) {
        this.display = false;
      }
      val.forEach((item) => {
        this.ids.push(item.id);
      });
    },

    add() {
      this.addOpen = true;
      this.title = "新增";
    },
    submit() {
      this.$refs.addForm.validate((valid) => {
        console.log(valid)
        if (valid) {
          this.axios.post("/assest/add").then((res) => {
            console.log(res);
            if(res.status == 200){
              this.reset()
            }
          });
        } else {
          this.$message({
            showClose: true,
            type: "error",
            message: "请将信息填写完整!",
          });
        }
      });
    },
    reset() {
      this.addOpen = false;
      this.form = {};
    },
    del() {
      this.$confirm("此操作将永久删除,是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    // echarts初始化
    echartInitialize() {
      const overview = echarts.init(document.getElementById("overview"));
      const option = {
        title: {
          text: "资金明细图",
        },
        tooltip: {
          trigger: "axis",
        },
        legend: {},
        toolbox: {
          feature: {
            saveAsImage: {},
          },
        },
        xAxis: {
          type: "category",
          boundaryGap: false,
          data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"],
        },
        yAxis: {},
        series: [
          {
            name: "销量",
            type: "line",
            data: [5, 20, 36, 10, 10, 20],
          },
        ],
      };

      overview.setOption(option);
      window.addEventListener("resize", () => {
        overview.resize();
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.button-wrap {
  padding: 20px;
}
.chart-btn {
  padding: 20px;
}
.tally-chart {
  #overview {
    width: 100%;
    height: 500px;
  }
}
</style>
