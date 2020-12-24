<!--
 * @Author: your name
 * @Date: 2020-12-22 20:15:41
 * @LastEditTime: 2020-12-22 20:46:12
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /renren-fast-vue/src/views/modules/project/noticeDialog.vue
-->
<template>
  <el-dialog
    title="消息详情"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-row>
      <el-col>
        <div class="notice-label">任务信息</div>
      </el-col>
    </el-row>
    <el-row>
      <el-col>
        <div v-html="detailInfo.noticeContent">
          {{ detailInfo.noticeContent }}
        </div>
      </el-col>
    </el-row>
    <div class="mod-notice-foot">
      <el-button @click="onback">关闭</el-button>
    </div>
  </el-dialog>
</template>
<script>
export default {
  data() {
    return {
      visible: false,
      noticeId: "",
      detailInfo: "",
    };
  },
  methods: {
    init(id) {
      this.noticeId = id;
      this.$http({
        url: this.$http.adornUrl("/project/monitor/noticeDetail"),
        method: "post",
        data: this.$http.adornData({
          noticeId: this.noticeId,
        }),
      }).then(({ data }) => {
        this.visible = true;
        if (data && data.code === 0) {
          this.detailInfo = data.records;
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    onback() {
      this.$http({
        url: this.$http.adornUrl("/project/monitor/updateNotice"),
        method: "post",
        data: this.$http.adornData({
          noticeId: this.noticeId,
        }),
      }).then(({ data }) => {
        this.visible = false;
        this.$emit("refreshDataList");
      });
    },
  },
};
</script>
<style lang="scss">
.notice-label {
  color: #606266;
  padding: 0 12px 0 0;
  font-size: 16px;
  font-weight: 500;
}
.mod-notice-foot {
  text-align: center;
  margin-top: 30px;
  margin-bottom: 30px;
}
</style>