<!--
 * @Author: your name
 * @Date: 2020-10-23 17:50:50
 * @LastEditTime: 2020-12-23 10:43:27
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /renren-fast-vue/src/views/common/home.vue
-->
<template>
  <div class="mod-home">
    <el-row :gutter="12">
      <el-col :span="12">
        <el-card shadow="hover" style="height: 400px">
          <div class="total-title">任务总量</div>
          <div class="total-value">123</div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card shadow="hover" style="height: 400px">
          <div id="J_chartPieBox" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
    <el-card class="" shadow="never" style="margin-top: 30px">
      <div slot="header" class="clearfix">
        <span>消息中心</span>
        <el-button
          style="float: right; padding: 3px 0"
          type="text"
          @click="$router.push({ name: 'project-notice' })"
          >更多</el-button
        >
      </div>
      <div class="notic-content">
        <el-table
          :data="dataList"
          v-loading="dataListLoading"
          border
          style="width: 100%"
        >
          <el-table-column
            prop="noticeTitle"
            header-align="center"
            align="center"
            label="消息名称"
          ></el-table-column>
          <el-table-column
            prop="noticeType"
            header-align="center"
            align="center"
            label="消息类型"
          >
            <template slot-scope="scope">
              <el-tag v-if="scope.row.noticeType === '0'" size="small"
                >通知</el-tag
              >
              <el-tag v-else size="small">预警</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            prop="status"
            header-align="center"
            align="center"
            label="消息状态"
          >
            <template slot-scope="scope">
              <el-tag v-if="scope.row.status === '0'" size="small">未读</el-tag>
              <el-tag v-else size="small">已读</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            prop="createTime"
            header-align="center"
            align="center"
            label="创建时间"
          ></el-table-column>
          <el-table-column header-align="center" align="center" label="操作">
            <template slot-scope="scope">
              <el-button
                type="text"
                size="small"
                @click="getNoticeDetail(scope.row.noticeId)"
                >查看详情</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-card>
    <notice-dialog
      v-if="noticeDetailVisible"
      ref="noticeDialog"
      @refreshDataList="getDataList"
    ></notice-dialog>
  </div>
</template>

<script>
import NoticeDialog from "../../views/modules/project/noticeDialog";
import echarts from "echarts";
export default {
  data() {
    return {
      dataListLoading: false,
      dataList: [],
      noticeDetailVisible: false,
      chartPie: null,
    };
  },
  components: {
    NoticeDialog,
  },
  created() {
    this.getNoticeData();
    this.initChartPie();
  },
  mounted() {},
  activated() {
    if (this.chartPie) {
      this.chartPie.resize();
    }
  },
  methods: {
    // 饼状图
    initChartPie() {
      var option = {
        backgroundColor: "#2c343c",
        title: {
          text: "Customized Pie",
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
            radius: "55%",
            center: ["50%", "50%"],
            data: [
              { value: 335, name: "直接访问" },
              { value: 310, name: "邮件营销" },
              { value: 274, name: "联盟广告" },
              { value: 235, name: "视频广告" },
              { value: 400, name: "搜索引擎" },
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
    getNoticeData() {
      this.$http({
        url: this.$http.adornUrl("/project/monitor/noticeList"),
        method: "post",
        data: this.$http.adornData(),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.list.list;
        } else {
          this.dataList = [];
        }
        this.dataListLoading = false;
      });
    },
    getNoticeDetail(id) {
      this.noticeDetailVisible = true;
      this.$nextTick(() => {
        this.$refs.noticeDialog.init(id);
      });
    },
  },
};
</script>

<style>
.mod-home {
  line-height: 1.5;
}
.total-frame {
  background-color: #83db0f;
  background-image: linear-gradient(#fd9d0d, #d8ae3c, #51bbad);
}
.total-title {
  position: relative;
  font-size: 16px;
  color: #909399;
  /* left: 70px;
    top: -50px; */
}
.total-value {
  position: relative;
  font-size: 18px;
  color: #606266;
  /* left: 70px;*/
  top: 10px;
}
</style>

