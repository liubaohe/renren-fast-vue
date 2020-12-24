<template>
  <div class="mod-feedback">
    <el-row>
      <el-form label-position="left" label-width="80px">
        <el-form-item label="任务标题">
          <el-input
            style="width: 400px"
            v-model="detailInfo.monitorTitle"
            readonly=""
          ></el-input>
        </el-form-item>
        <el-form-item label="下发组织">
          <el-input
            style="width: 400px"
            v-model="detailInfo.deptName"
            readonly
          ></el-input>
        </el-form-item>
      </el-form>
    </el-row>
    <el-row>
      <el-col>
        <div class="feedback-label">任务信息</div>
      </el-col>
    </el-row>
    <el-row>
      <el-col>
        <div v-html="detailInfo.monitorTask">
          {{ detailInfo.monitorTask }}
        </div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="4">
        <div class="feedback-label" style="margin-bottom: 10px">
          任务完成反馈
        </div>
      </el-col>
      <el-col>
        <div class="mod-demo-ueditor">
          <script
            :id="ueId"
            class="ueditor-box"
            type="text/plain"
            style="width: 100%; height: 260px"
          ></script>
        </div>
      </el-col>
    </el-row>
    <div class="mod-feedback-foot">
      <el-button type="primary" @click="submitFeeBack">提交</el-button>
      <el-button @click="onback">取消</el-button>
    </div>
  </div>
</template>
<script>
import ueditor from "ueditor";
export default {
  data() {
    return {
      ue: null,
      ueId: `J_ueditorBox_${new Date().getTime()}`,
      monitorId: "",
      detailInfo: "",
      monitorFeedback:''
    };
  },
  created() {
    this.monitorId = this.$route.query.monitorId;
  },
  mounted() {
    this.ue = ueditor.getEditor(this.ueId, {
      // serverUrl: '', // 服务器统一请求接口路径
      zIndex: 3000,
    });
    this.getDetailsInfo();
  },
  methods: {
    onback() {
      this.$router.push({ name: "project-check" });
    },
    getDetailsInfo() {
      this.$http({
        url: this.$http.adornUrl("/project/monitor/detail"),
        method: "post",
        data: this.$http.adornData({
          monitorId: this.monitorId,
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.detailInfo = data.records;
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    submitFeeBack() {
       var monitorFeedback = this.ue.getContent();
      if (
        monitorFeedback == null ||
        monitorFeedback == "" ||
        monitorFeedback == undefined
      ) {
        this.$message.info("请输入任务反馈结果");
        return;
      }
      this.$http({
        url: this.$http.adornUrl("/project/monitor/feedBack"),
        method: "post",
        data: this.$http.adornData({
            "monitorId":this.monitorId,
            "monitorFeedBack":monitorFeedback
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message.info(data.msg);
          this.$router.replace({ name: "project-check" });
        } else {
          this.$message.error(data.msg);
        }
      });
    },
  },
};
</script>

<style lang="scss">
.feedback-label {
  color: #606266;
  padding: 0 12px 0 0;
}
.mod-feedback-foot {
  text-align: center;
  margin-top: 30px;
  margin-bottom: 30px;
}
</style>