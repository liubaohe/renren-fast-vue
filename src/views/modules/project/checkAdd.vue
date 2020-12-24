
<template>
  <div class="mod-monitor-editor">
    <el-form ref="form" :model="form" label-width="100px">
      <el-form-item label="任务标题：">
        <el-input
          v-model="form.monitorTitle"
          class="form-control"
          placeholder="请输入任务标题"
        ></el-input>
      </el-form-item>
      <el-form-item label="下发组织：">
        <el-select v-model="form.region" placeholder="请选择下发组织">
          <el-option
            :label="item.deptName"
            :value="item"
            v-for="item in deptList"
            :key="item.deptId"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="任务信息："> </el-form-item>
    </el-form>
    <div class="mod-demo-ueditor">
      <script
        :id="ueId"
        class="ueditor-box"
        type="text/plain"
        style="width: 100%; height: 260px"
      ></script>
    </div>
    <div class="mod-monitor-foot">
      <el-button type="primary" @click="onSubmit">立即创建</el-button>
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
      ueContent: "",
      deptList:[],
      form: {
        monitorTitle: "",
        region: "",
      },
    };
  },
  mounted() {
      this.getDeptList();
    this.ue = ueditor.getEditor(this.ueId, {
      // serverUrl: '', // 服务器统一请求接口路径
      zIndex: 3000,
    });
  },
  methods: {
    getDeptList() {
        this.$http({
        url: this.$http.adornUrl("/project/monitor/deptList"),
        method: "post",
        data: this.$http.adornData(),
      }).then(({ data }) => {
        if (data && data.code === 0) {
            this.deptList = data.list;
        } 
      });
    },
    onback() {
      this.$router.replace({ name: "project-check" });
    },
    //参数提交
    onSubmit() {
      var submitParams = {};
      submitParams.monitorTitle = this.form.monitorTitle;
      if (
        submitParams.monitorTitle == null ||
        submitParams.monitorTitle == "" ||
        submitParams.monitorTitle == undefined
      ) {
        this.$message.info("请输入任务标题");
        return null;
      }
      submitParams.deptId = this.form.region.deptId;
      if (
        submitParams.deptId == null ||
        submitParams.deptId == "" ||
        submitParams.deptId == undefined
      ) {
        this.$message.info("请选择下发组织");
        return;
      }
      submitParams.monitorTask = this.ue.getContent();
      if (
        submitParams.monitorTask == null ||
        submitParams.monitorTask == "" ||
        submitParams.monitorTask == undefined
      ) {
        this.$message.info("请输入任务信息");
        return;
      }
      this.$http({
        url: this.$http.adornUrl("/project/monitor/add"),
        method: "post",
        data: this.$http.adornData(submitParams),
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
.mod-demo-ueditor {
  position: relative;
  z-index: 510;
}
.form-control {
  width: 300px;
}
.mod-monitor-foot {
  text-align: center;
  margin-top: 30px;
  margin-bottom: 30px;
}
</style>
