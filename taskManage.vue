<!--
  *@description Vue Files Modules
  *@fileName Info.vue
  *@author leejie
  *@date 2024/06/27 13:58:14
!-->
<template>
  <div id="mainbox" class="page-wrap">
    <el-form ref="form" :inline="true" :model="queryForm" size="small" label-width="100px">
      <el-row>
        <el-col :span="6">
          <el-form-item label="任务名称">
            <el-input v-model="queryForm.taskName" style="width: 220px;" />
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="任务状态">
            <el-select v-model="queryForm.taskState" placeholder="请选择">
              <el-option v-for="item in taskStateOptions" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item label="创建时间">
            <el-date-picker v-model="dateRange" type="daterange" format="yyyy-MM-dd" value-format="yyyy-MM-dd" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" @change="handleDateChange" />
          </el-form-item>
        </el-col>
        <el-col :span="4">
          <el-form-item>
            <el-button type="primary" @click="queryData">查询</el-button>
            <el-button @click="resetQuery">重置</el-button>
            <el-button type="success" @click="operateTask('run')">运行</el-button>
            <el-button type="danger" @click="operateTask('pause')">暂停</el-button>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <el-table ref="taskTable" :data="tableData" stripe size="small" height="calc(100% - 105px)" max-height="calc(100% - 105px)" style="width: 100%" @selection-change="selectionLineChangeHandle">
      <el-table-column type="selection" width="55" />
      <el-table-column type="index" label="序号" width="50" align="center" />
      <el-table-column prop="taskName" label="任务名称" align="center" />
      <el-table-column prop="taskState" label="任务状态" align="center">
        <template v-slot="scope">
          <span>{{ scope.row.taskState === '1' ? '运行' : '暂停' }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="trigGap" label="触发间隔" align="center" />
      <el-table-column prop="trigTimes" label="触发总次数" align="center" />
      <!-- <el-table-column prop="startTime" label="开始时间" align="center"></el-table-column> -->
      <el-table-column prop="createTime" label="创建时间" align="center" />
      <el-table-column align="center" label="操作" width="200">
        <template v-slot="scope">
          <el-button @click.prevent="runRecord(scope.row.id)" type="text" size="small">
            运行记录
          </el-button>
          <el-button @click.prevent="historyRecord(scope.row.id)" type="text" size="small">
            历史记录
          </el-button>
          <el-button @click.prevent="copyTask(scope.$index, scope.row)" type="text" size="small">
            复制
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="pagination-wrap">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryForm.page" :page-sizes="[10, 20, 50, 100]" :page-size="queryForm.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total" />
    </div>
    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible">
      <div v-if="dialogTitle === '运行记录'" style="display: flex; justify-content: center; min-height: 400px;">
        <el-steps :active="taskRunList.length" direction="vertical" align-center>
          <el-step v-for="step in taskRunList" :key="step.id" :title="step.date" :description="step.name" />
        </el-steps>
      </div>
      <div v-else>
        <el-table :data="taskHistory" height="500px">
          <el-table-column property="operateName" label="操作名称" />
          <el-table-column property="operateTime" label="操作时间" />
          <el-table-column property="operater" label="操作人" />
        </el-table>
      </div>
    </el-dialog>
  </div>
</template>

<script>
const totalTable = [
  { id: 1, taskName: '微件使用分析定时任务', taskState: '1', trigGap: '2次/天', trigTimes: 3, startTime: '', createTime: '2023-01-23', isShow: true },
  { id: 2, taskName: '定时上报微件统计数据', taskState: '1', trigGap: '1次/30天', trigTimes: 1, startTime: '', createTime: '2023-01-25', isShow: true },
  { id: 3, taskName: '定时上报分辨率统计数据', taskState: '1', trigGap: '1次/10天', trigTimes: 1, startTime: '', createTime: '2023-02-02', isShow: true },
  { id: 4, taskName: '检测统计任务', taskState: '2', trigGap: '3次/20天', trigTimes: 0, startTime: '', createTime: '2023-03-13', isShow: true },
  { id: 5, taskName: '实时监控任务', taskState: '1', trigGap: '1次/2小时', trigTimes: 10, startTime: '', createTime: '2023-03-25', isShow: true },
  { id: 6, taskName: '用户反馈收集与处理', taskState: '1', trigGap: '1次/3天', trigTimes: 2, startTime: '', createTime: '2023-05-09', isShow: true },
  { id: 7, taskName: '客户关系维护计划制定', taskState: '1', trigGap: '1次/10天', trigTimes: 1, startTime: '', createTime: '2023-05-18', isShow: true },
  { id: 8, taskName: '市场调研报告撰写与分析', taskState: '2', trigGap: '1次/7天', trigTimes: 1, startTime: '', createTime: '2023-05-30', isShow: true },
  { id: 9, taskName: '销售团队业绩考核与激励', taskState: '2', trigGap: '1次/3天', trigTimes: 3, startTime: '', createTime: '2023-05-31', isShow: true },
  { id: 10, taskName: '更新网站内容', taskState: '1', trigGap: '1次/1小时', trigTimes: 12, startTime: '', createTime: '2023-06-02', isShow: true },
  { id: 11, taskName: '设计产品原型', taskState: '1', trigGap: '2次/3天', trigTimes: 2, startTime: '', createTime: '2023-06-12', isShow: true },
  { id: 12, taskName: '参与培训课程', taskState: '2', trigGap: '5次/天', trigTimes: 10, startTime: '', createTime: '2023-08-08', isShow: true },
  { id: 13, taskName: '提升团队效率', taskState: '1', trigGap: '1次/10天', trigTimes: 1, startTime: '', createTime: '2023-08-29', isShow: true },
  { id: 14, taskName: '分析销售数据', taskState: '2', trigGap: '1次/15天', trigTimes: 5, startTime: '', createTime: '2023-09-30', isShow: true },
  { id: 15, taskName: '完善客户服务流程', taskState: '1', trigGap: '1次/30天', trigTimes: 1, startTime: '', createTime: '2023-10-22', isShow: true },
  { id: 16, taskName: '发布社交媒体内容', taskState: '1', trigGap: '1次/3天', trigTimes: 2, startTime: '', createTime: '2023-10-24', isShow: true },
  { id: 17, taskName: '拍摄宣传视频', taskState: '1', trigGap: '1次/12天', trigTimes: 3, startTime: '', createTime: '2023-11-09', isShow: true },
  { id: 18, taskName: '改善工作流程', taskState: '1', trigGap: '3次/5天', trigTimes: 1, startTime: '', createTime: '2023-11-10', isShow: true },
  { id: 19, taskName: '协助招聘新员工', taskState: '1', trigGap: '1次/30天', trigTimes: 4, startTime: '', createTime: '2023-12-22', isShow: true },
  { id: 20, taskName: '进行市场调研', taskState: '2', trigGap: '2次/3天', trigTimes: 1, startTime: '', createTime: '2023-12-29', isShow: true },
  { id: 21, taskName: '项目进度跟进与协调', taskState: '2', trigGap: '3次/天', trigTimes: 6, startTime: '', createTime: '2023-12-31', isShow: true },
  { id: 22, taskName: '竞品分析报告编写', taskState: '2', trigGap: '1次/3天', trigTimes: 2, startTime: '', createTime: '2024-01-01', isShow: true },
  { id: 23, taskName: '网站内容更新与维护', taskState: '1', trigGap: '1次/15天', trigTimes: 1, startTime: '', createTime: '2024-02-28', isShow: true },
  { id: 24, taskName: '跨部门协作任务对接', taskState: '1', trigGap: '2次/3天', trigTimes: 3, startTime: '', createTime: '2024-03-01', isShow: true },
  { id: 25, taskName: '库存盘点与异常处理', taskState: '1', trigGap: '1次/30天', trigTimes: 5, startTime: '', createTime: '2024-03-15', isShow: true },
  { id: 26, taskName: '项目文档整理与归档', taskState: '2', trigGap: '1次/15天', trigTimes: 1, startTime: '', createTime: '2024-03-21', isShow: true }
];
export default {
  name: 'TaskManage',
  data() {
    return {
      dateRange: [],
      queryForm: {
        taskName: '',
        taskState: '',
        page: 1,
        pageSize: 20
      },
      tableData: [],
      total: 0,
      taskStateOptions: [
        { label: '运行', value: '1' },
        { label: '暂停', value: '2' }
      ],
      tableChecked: [],
      copyData: {},
      taskRunList: [
        { id: 1, name: '任务运行', date: '2023-02-21 08:08:06' },
        { id: 2, name: '任务运行', date: '2023-02-28 18:23:05' },
        { id: 3, name: '任务运行', date: '2023-03-09 10:54:08' },
        { id: 4, name: '任务运行', date: '2023-04-21 14:12:09' },
        { id: 5, name: '任务运行', date: '2023-05-18 04:35:02' }
      ], // 任务运行记录
      taskHistory: [
        { id: 1, operateName: '任务运行', operateTime: '2023-01-12 10:23:34', operater: '超级管理员' },
        { id: 2, operateName: '任务暂停', operateTime: '2023-02-22 01:12:56', operater: 'admin' },
        { id: 3, operateName: '任务运行', operateTime: '2023-05-05 09:43:37', operater: '张三' },
        { id: 4, operateName: '任务复制', operateTime: '2023-05-14 05:23:04', operater: '王五' },
        { id: 5, operateName: '任务暂停', operateTime: '2023-06-08 08:08:05', operater: '李四' },
        { id: 6, operateName: '任务暂停', operateTime: '2023-07-15 14:56:43', operater: '马六' },
        { id: 7, operateName: '任务运行', operateTime: '2023-09-30 20:59:59', operater: '超级管理员' },
        { id: 8, operateName: '任务运行', operateTime: '2023-10-22 12:22:21', operater: '超级管理员' },
        { id: 9, operateName: '任务复制', operateTime: '2023-11-11 23:23:33', operater: 'admin' },
        { id: 10, operateName: '任务暂停', operateTime: '2023-12-22 09:49:42', operater: '赵德柱' }
      ], // 任务历史记录
      dialogTitle: '',
      dialogVisible: false
    };
  },
  mounted() {
    this.tableData = totalTable.slice(0, 20);
    this.total = totalTable.length;
  },
  // 方法实现
  methods: {
    queryData() {
      this.tableData = [];
      totalTable.forEach(item => {
        // 筛选条件 任务状态有值时
        if (this.queryForm.taskState) {
          // 筛选条件选择了时间
          if (this.dateRange[0]) {
            if (item.taskName.includes(this.queryForm.taskName) && item.taskState === this.queryForm.taskState && this.getDateTimestamp(item.createTime) >= this.getDateTimestamp(this.dateRange[0]) && this.getDateTimestamp(item.createTime) <= this.getDateTimestamp(this.dateRange[1])) {
              this.tableData.push(item);
            }
          } else {
            if (item.taskName.includes(this.queryForm.taskName) && item.taskState === this.queryForm.taskState) {
              this.tableData.push(item);
            }
          }
        } else { // 筛选条件 任务状态没值时
          // 筛选条件选择了时间
          if (this.dateRange[0]) {
            if (item.taskName.includes(this.queryForm.taskName) && this.getDateTimestamp(item.createTime) >= this.getDateTimestamp(this.dateRange[0]) && this.getDateTimestamp(item.createTime) <= this.getDateTimestamp(this.dateRange[1])) {
              this.tableData.push(item);
            }
          } else {
            if (item.taskName.includes(this.queryForm.taskName)) {
              this.tableData.push(item);
            }
          }
        }
      });
      this.total = this.tableData.length;
    },
    resetQuery() {
      this.dateRange = [];
      this.queryForm = {
        taskName: '',
        taskState: '',
        page: 1,
        pageSize: 20
      };
      this.queryForm.page = 1;
      this.tableData = totalTable.slice(0, 20);
      this.total = totalTable.length;
    },
    handleSizeChange(size) {
      this.queryForm.pageSize = size;
      this.queryForm.page = 1;
      this.tableData = totalTable.slice(0, size);
    },
    handleCurrentChange(page) {
      this.queryForm.page = page;
      console.log(page, this.queryForm.pageSize);
      this.tableData = totalTable.slice((page - 1) * this.queryForm.pageSize, page * this.queryForm.pageSize);
    },
    handleDateChange(val) {
      console.log(val, this.dateRange);
    },
    // 选择行数据
    selectionLineChangeHandle(val) {
      console.log(val);
      this.tableChecked = val;
    },
    // 运行任务、暂停任务
    operateTask(type) {
      if (!this.tableChecked.length) {
        this.$message.warning('请选择要操作的数据');
        return;
      }
      this.tableChecked.forEach(item => {
        this.tableData.forEach(obj => {
          if (item.id === obj.id) {
            obj.taskState = type === 'run' ? '1' : '2';
          }
        });
      });
      this.$refs.taskTable.clearSelection();
      this.$message.success('操作成功');
    },
    // 运行记录
    runRecord(id) {
      this.dialogTitle = '运行记录';
      this.dialogVisible = true;
    },
    // 历史记录
    historyRecord(id) {
      this.dialogTitle = '历史记录';
      this.dialogVisible = true;
    },
    // 复制任务
    copyTask(idx, data) {
      const copyData = JSON.parse(JSON.stringify(data));
      copyData.id = data.id + '01';
      copyData.taskName = data.taskName + ' 副本';
      this.tableData.splice(idx, 0, copyData);
      this.tableData.pop();
      totalTable.splice(((this.queryForm.page - 1) * this.queryForm.pageSize) + idx, 0, copyData);
      this.total = totalTable.length;
      this.$message.success('复制成功');
    },
    // 获取指定日期的时间戳
    getDateTimestamp(date) {
      return new Date(date + ' 00:00:00').getTime();
    }
  }
};
</script>

<style lang='scss' scoped>
.page-wrap {
  height: 100%;
}
.pagination-wrap {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}
::v-deep .el-dialog__body {
  padding-top: 30px !important;
}
::v-deep .el-steps--vertical {
  height: auto;
}
</style>
