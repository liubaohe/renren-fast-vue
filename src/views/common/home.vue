<template>
  <div class="mod-home">
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card>
          <div id="J_chartPieBox" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <div class="chart-box">
            <el-row :gutter="20">
              <el-col :span="12">
                <div class="total-frame">
                  <img :src="img_no_finish" class="total-icon" />
                  <div class="total-title">总未完成数量</div>
                  <div class="total-value">{{noFinishTotal}}</div>
                </div>
              </el-col>
               <el-col :span="12">
                <div class="total-frame">
                  <img :src="img_time_out" class="total-icon" />
                  <div class="total-title">总超时数量</div>
                  <div class="total-value">{{timeOutTotal}}</div>
                </div>
              </el-col>
            </el-row>
            <el-row>
              <el-col>
                <div class="total-frame">
                  <img :src="img_finish" class="total-icon" />
                  <div class="total-title">完成总数量</div>
                  <div class="total-value">{{finshTotal}}</div>
                </div>
              </el-col>
            </el-row>
            <el-row>
              <el-col>
                <div class="total-frame">
                  <img :src="img_total" class="total-icon" />
                  <div class="total-title">总任务量</div>
                  <div class="total-value">{{taskTotal}}</div>
                </div>
              </el-col>
            </el-row>
          </div>
        </el-card>
      </el-col>
      <el-col :span="24">
        <el-card>
          <div id="J_chartLineBox" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import echarts from "echarts";
import img_total from "@/assets/img/total_num.png"
import img_time_out from "@/assets/img/tome_out.png"
import img_finish from "@/assets/img/finish.png"
import img_no_finish from "@/assets/img/no_finsh_icon.png"
export default {
  data() {
    return {
      chartLine: null,
      chartPie: null,
      totalData: null,
      img_total,
      img_time_out,
      img_finish,
      img_no_finish,
      taskTotal:0,
      finshTotal:0,
      noFinishTotal:0,
      timeOutTotal:0,
    };
  },
  mounted() {
    this.getTotalData();
  },
  activated() {
    // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
    if (this.chartLine) {
      this.chartLine.resize();
    }
    if (this.chartPie) {
      this.chartPie.resize();
    }
  },
  methods: {
    getTotalData() {
      this.$http({
        url: this.$http.adornUrl("/project/monitor/total"),
        method: "post",
        data: this.$http.adornData(),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.totalData = data.records;
          this.taskTotal =data.records.total;
          this.finshTotal=data.records.finish;
          this.noFinishTotal=data.records.noFinish;
          this.timeOutTotal=data.records.timeOut;
          this.initChartPie(this.totalData);
          this.initChartLine(data.records.line);
        }
      });
    },
    // 折线图
    initChartLine(lineData) {
      var option = {
        title: {
          text: "近一周任务完成情况",
        },
        tooltip: {
          trigger: "axis",
        },
        legend: {
          data: ["已完成", "未完成", "超时"],
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        toolbox: {
          feature: {
            saveAsImage: {},
          },
        },
        xAxis: {
          type: "category",
          boundaryGap: false,
          data: lineData.xAxis,
        },
        yAxis: {
          type: "value",
        },
        series: [
          {
            name: "已完成",
            type: "line",
            data: lineData.finish,
          },
          {
            name: "未完成",
            type: "line",
            data: lineData.noFinish,
          },
          {
            name: "超时",
            type: "line",
            data: lineData.timeOut,
          },
        ],
      };
      this.chartLine = echarts.init(document.getElementById("J_chartLineBox"));
      this.chartLine.setOption(option);
      window.addEventListener("resize", () => {
        this.chartLine.resize();
      });
    },
    // 饼状图
    initChartPie(totalData) {
      var option = {
        backgroundColor: "#2c343c",
        title: {
          text: "总任务完成情况",
          left: "center",
          top: 20,
          textStyle: {
            color: "#ccc",
          },
        },
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b} : {c} ({d}%)",
        },
        visualMap: {
          show: false,
          min: 80,
          max: 600,
          inRange: {
            colorLightness: [0, 1],
          },
        },
        series: [
          {
            name: "访问来源",
            type: "pie",
            radius: ["40%", "60%"],
            center: ["50%", "50%"],
            data: [
              { value: totalData.noFinish, name: "未完成" },
              { value: totalData.finish, name: "已完成" },
              { value: totalData.timeOut, name: "超时" },
            ].sort(function (a, b) {
              return a.value - b.value;
            }),
            roseType: "radius",
            label: {
              normal: {
                textStyle: {
                  color: "rgba(255, 255, 255, 0.3)",
                },
              },
            },
            labelLine: {
              normal: {
                lineStyle: {
                  color: "rgba(255, 255, 255, 0.3)",
                },
                smooth: 0.2,
                length: 10,
                length2: 20,
              },
            },
            itemStyle: {
              normal: {
                color: "#c23531",
                shadowBlur: 200,
                shadowColor: "rgba(0, 0, 0, 0.5)",
              },
            },
            animationType: "scale",
            animationEasing: "elasticOut",
            animationDelay: function (idx) {
              return Math.random() * 200;
            },
          },
        ],
      };
      this.chartPie = echarts.init(document.getElementById("J_chartPieBox"));
      this.chartPie.setOption(option);
      window.addEventListener("resize", () => {
        this.chartPie.resize();
      });
    },
  },
};
</script>

<style lang="scss">
.mod-home {
  line-height: 1.5;
  > .el-row {
    margin-top: -10px;
    margin-bottom: -10px;
    .el-col {
      padding-top: 10px;
      padding-bottom: 10px;
    }
  }
  .chart-box {
    min-height: 400px;
  }
}
 .total-frame {
    border: 1px solid #DCDFE6;
    padding: 20px;
    height: 100px;
  }
  .total-icon {
    color: #409EFF;
    width: 60px;
    height: 60px;
  }
  .total-title {
    position: relative;
    font-size: 16px;
    color: #909399;
    left: 80px;
    top: -60px;
  }
  .total-value {
    position: relative;
    font-size: 18px;
    color: #606266;
    left: 80px;
    top: -50px;
  }

</style>
