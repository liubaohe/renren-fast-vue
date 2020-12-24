<template>
  <div class="mod-notice">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item label="消息状态">
        <el-select v-model="dataForm.status" placeholder="请选择消息类型">
          <el-option label="未读" value="0"></el-option>
          <el-option label="已读" value="1"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="消息类型">
        <el-select v-model="dataForm.noticeType" placeholder="请选择消息类型">
          <el-option label="通知" value="0"></el-option>
          <el-option label="告警" value="1"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
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
          <el-tag v-if="scope.row.noticeType === '0'" size="small" >通知</el-tag>
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
          <el-tag v-if="scope.row.status === '0'" size="small" >未读</el-tag>
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
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
    <notice-dialog
      v-if="noticeDetailVisible"
      ref="noticeDialog"
      @refreshDataList="getDataList"
    ></notice-dialog>
  </div>
</template>

<script>
import NoticeDialog from "./noticeDialog";
export default {
  data() {
    return {
      dataForm: {
        status: "0",
        noticeType: "",
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      noticeDetailVisible: false,
    };
  },
  components: {
    NoticeDialog,
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/project/monitor/noticeList"),
        method: "post",
        data: this.$http.adornData({
          page: this.pageIndex,
          limit: this.pageSize,
          status: this.dataForm.status,
          noticeType: this.dataForm.noticeType,
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.list.list;
          this.totalPage = data.list.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
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
