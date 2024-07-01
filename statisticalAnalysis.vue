<!--
  *@description Vue Files Modules
  *@fileName Info.vue
  *@author leejie
  *@date 2024/06/27 13:58:14
!-->
<template>
  <div id="mainbox" class="page-wrap">
    <!-- <div class="upload-btn">
      <el-upload action="" :limit="1" :on-change="handleSuccess" :show-file-list="false" :auto-upload="false" accept=".xls,.xlsx">
        <el-button size="small" type="primary">点击上传</el-button>
      </el-upload>
    </div> -->
    <div class="page-top">
      <div class="top-left">
        <div class="chart-title">多微应用分析统计-菜单调用占比</div>
        <div ref="menuUseZb" style="width: 100%;height: calc(100% - 40px);" />
      </div>
      <div class="top-right">
        <div class="chart-title">多微应用分析统计-菜单调用统计</div>
        <div ref="menuUseTj" style="width: 100%;height: calc(100% - 40px);" />
      </div>
    </div>
    <div class="page-bottom">
      <div class="chart-title">多微应用分析统计-耗时统计</div>
      <div ref="useTimeTj" style="width: 100%;height: calc(100% - 40px);" />
    </div>
  </div>
</template>

<script>
import echarts from 'echarts';
import XLSX from 'xlsx';
export default {
  name: 'statisticalAnalysis',
  data() {
    return {
      menuUseZbChart: null,
      menuUseTjChart: null,
      useTimeTjChart: null
    };
  },
  mounted() {
    this.initMenuUseZbEchart();
    this.initMenuUseTjEchart();
    this.initUseTimeTjEchart();
  },
  // 方法实现
  methods: {
    // 菜单调用占比图表初始化
    initMenuUseZbEchart() {
      this.$nextTick(() => {
        this.menuUseZbChart = echarts.init(this.$refs.menuUseZb, null, { renderer: 'canvas', devicePixelRatio: 3 });
        // 把配置和数据放这里
        this.menuUseZbChart.setOption({
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            }
          },
          color: ['#4169E1'], // #32CD32
          grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
          },
          xAxis: [
            {
              type: 'category',
              data: ['工作量', '巡视管理', '项目计划管理', '隐患管理', '试验计划'],
              axisTick: {
                alignWithLabel: true
              }
            }
          ],
          yAxis: [
            {
              type: 'value',
              name: '单位: %',
              axisLine: {
                show: false
              },
              axisTick: {
                show: false
              },
              min: 0,
              max: 100
            }
          ],
          series: [
            {
              name: '菜单调用占比',
              type: 'bar',
              barWidth: 20,
              data: [82, 52, 20, 34, 90, 33, 22]
            }
          ]
        }, true);
        // this.menuUseZbChart.on('click', params => {
        //   // 这个params可以获取你要的图中的当前点击的项的参数
        //   this.$emit('callback', params);
        // });
      });
    },
    // 菜单调用统计图表初始化
    initMenuUseTjEchart() {
      this.$nextTick(() => {
        this.menuUseTjChart = echarts.init(this.$refs.menuUseTj, null, { renderer: 'canvas', devicePixelRatio: 3 });
        // 把配置和数据放这里
        this.menuUseTjChart.setOption({
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            }
          },
          color: ['#4169E1', '#32CD32'],
          legend: {
            right: 20
          },
          grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
          },
          xAxis: {
            type: 'category',
            data: ['电源用户电压', '数据操作', 'ERP监控接口', '运行监控']
          },
          yAxis: {
            type: 'value',
            name: '单位: 次',
            axisLine: {
              show: false
            },
            axisTick: {
              show: false
            }
          },
          series: [
            {
              name: '成功次数',
              type: 'bar',
              barWidth: 15,
              data: [203, 489, 34, 970]
            },
            {
              name: '失败次数',
              type: 'bar',
              barWidth: 15,
              data: [5, 3, 1, 2]
            }
          ]
        }, true);
      });
    },
    // 耗时统计图表初始化
    initUseTimeTjEchart() {
      this.$nextTick(() => {
        this.useTimeTjChart = echarts.init(this.$refs.useTimeTj, null, { renderer: 'canvas', devicePixelRatio: 3 });
        // 把配置和数据放这里
        this.useTimeTjChart.setOption({
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            }
          },
          color: ['#4169E1', '#32CD32'],
          legend: {
            right: 20
          },
          grid: {
            left: '3%',
            right: '2%',
            bottom: '3%',
            containLabel: true
          },
          xAxis: {
            type: 'category',
            data: ['运行监控', '业务节点', '权限操作', '电源用户电压', '数据操作', 'ERP监控接口', '微应用', '组织人员', '用户授权', '运行日志']
          },
          yAxis: {
            type: 'value',
            name: '单位: ms',
            axisLine: {
              show: false
            },
            axisTick: {
              show: false
            }
          },
          series: [
            {
              name: '最大耗时',
              type: 'bar',
              barWidth: 15,
              data: [203, 489, 34, 970, 121, 234, 45, 94, 188, 78]
            },
            {
              name: '平均耗时',
              type: 'bar',
              barWidth: 15,
              data: [203, 489, 31, 121, 34, 223, 12, 44, 62, 65]
            }
          ]
        }, true);
      });
    },
    // 导入文件
    handleSuccess(response, file, fileList) {
      // 导入状态和文件信息
      const that = this;
      // 构建fileReader对象
      const fileReader = new FileReader();
      // 该事件为读取完成时触发
      fileReader.onload = function (ev) {
        try {
          const data = ev.target.result;
          const workbook = XLSX.read(data, { type: 'buffer' });
          console.log(workbook);
          const listNew = XLSX.utils.sheet_to_json(workbook.Sheets['系统集成完善'], { header: 1 });
          console.log(listNew);
        } catch (e) {
          that.$message({ message: '文件类型不正确', type: 'warning' });
        }
      };
      // 读取数据
      fileReader.readAsArrayBuffer(file[0].raw);
    }
  }
};
</script>

<style lang='scss' scoped>
.page-wrap {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
}
.upload-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 5;
}
.page-top {
  width: 100%;
  height: calc((100% - 20px) * 0.5);
  margin-bottom: 20px;
  display: flex;
  justify-content: space-between;
}
.top-right, .top-left {
  width: calc((100% - 20px) * 0.5);
  height: 100%;
  background: #fff;
  border-radius: 5px;
}
.page-bottom {
  width: 100%;
  height: calc((100% - 20px) * 0.5);
  background: #fff;
  border-radius: 5px;
}
.chart-title {
  font-size: 16px;
  font-weight: 500;
  line-height: 25px;
  padding: 15px 0 0 20px;
}
</style>
