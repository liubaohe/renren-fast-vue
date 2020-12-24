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
      <el-col>
        <div class="feedback-label">任务反馈信息</div>
      </el-col>
    </el-row>
    <el-row>
      <el-col>
        <div v-html="detailInfo.monitorFeedBack">
          {{ detailInfo.monitorFeedBack }}
        </div>
      </el-col>
    </el-row>
    <div class="mod-feedback-foot">
      <el-button @click="onback">返回</el-button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      monitorId: "",
      detailInfo: "",
    };
  },
  created() {
    this.monitorId = this.$route.query.monitorId;
  },
  mounted() {
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