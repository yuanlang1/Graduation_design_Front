<template>
  <view class="overview-container">
    <!-- 顶部统计卡片行 -->
    <view class="top-row">
      <view class="info-box" v-for="(item, index) in topData" :key="index">
        <text class="info-title">{{ item.title }}</text>
        <text class="info-value">{{ item.value }}</text>
      </view>
    </view>

    <!-- 年级课堂专注度柱状图 -->
    <view class="chart-row">
      <view class="chart-header">
        <text class="chart-title">年级课堂专注度</text>
      </view>
      <view class="charts-box">
        <qiun-data-charts
          type="column"
          :chartData="chartData"
          :canvas2d="true"
          canvasId="barChart"
          background="none"
          :ontouch="true"
          :opts="chartOpts"
        />
      </view>
      <text v-if="!chartDataLoaded">正在加载图表数据...</text>
    </view>

    <!-- 上堂课课堂专注度前10名班级 -->
    <view class="table-row">
      <text class="table-title">上堂课课堂专注度前10名班级</text>
      <view class="table">
        <!-- 表头 -->
        <view class="table-header">
          <text class="table-th">班级</text>
          <text class="table-th">任课教师</text>
          <text class="table-th">得分</text>
        </view>
        <!-- 表体 -->
        <view class="table-body" v-for="(row, idx) in topClasses" :key="idx">
          <text class="table-td">{{ row.className }}</text>
          <text class="table-td">{{ row.teacher }}</text>
          <text class="table-td">{{ row.score }}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  name: 'Overview',
  data() {
    return {
      topData: [
        { title: '全校班级数', value: 0 },
        { title: '全校正在上课班级数', value: 0 },
        { title: '当前课堂专注度', value: '无' },
      ],
      chartData: {
        categories: ['一年级', '二年级', '三年级', '四年级', '五年级', '六年级'],
        series: [
          {
            name: '年级专注度',
            data: [0.6, 0.5, 0.7, 0.6, 0.5, 0.7],
            color: '#1E90FF',
          },
        ],
      },
      chartDataLoaded: false,
      chartOpts: {
        yAxis: {
          min: 0,
          max: 1,
          format: (val) => val.toFixed(1), // 格式化纵轴值
        },
        xAxis: {
          disabled: false,
          axisLine: false,
          axisLineColor: '#EEEEEE',
          calibration: false,
          fontColor: '#999999',
          fontSize: 12,
          rotateLabel: false,
          labelCount: 6,
          boundaryGap: 'justify',
          disableGrid: true,
          gridColor: '#CCCCCC',
          gridType: 'dash',
          dashLength: 4,
          gridEval: 1,
        },
        legend: {
          show: false, // 隐藏图例，因为只有一组数据
        },
        dataLabel: false, // 隐藏数据标签
        dataPointShape: false, // 隐藏数据点形状
        animation: true, // 启用动画
        padding: [20, 20, 10, 5], // 图表内边距
      },
      topClasses: [],
    };
  },
  onLoad() {
    this.fetchTopStats();
    this.fetchChartData();
    this.fetchTop10Classes();
  },
  methods: {
    // 获取顶部统计数据
    async fetchTopStats() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/home/top_stats/',
          method: 'GET',
        });
        if (res.data.code === 200) {
          const { totalClasses, activeClasses, focusLevel } = res.data.data;
          this.topData = [
            { title: '全校班级数', value: totalClasses },
            { title: '全校正在上课班级数', value: activeClasses },
            { title: '当前课堂专注度', value: focusLevel },
          ];
        } else {
          uni.showToast({ title: '获取顶部统计失败', icon: 'none' });
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' });
      }
    },

    // 获取图表数据
    async fetchChartData() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/home/chart_data/',
          method: 'GET',
        });
        if (res.data.code === 200) {
          this.chartData = res.data.data;
          this.chartDataLoaded = true;
        } else {
          uni.showToast({ title: '获取图表数据失败', icon: 'none' });
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' });
      }
    },

    // 获取上堂课前10名班级
    async fetchTop10Classes() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/home/top10_classes/',
          method: 'GET',
        });
        if (res.data.code === 200) {
          this.topClasses = res.data.data;
        } else {
          uni.showToast({ title: '获取前10名班级失败', icon: 'none' });
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' });
      }
    },
  },
};
</script>

<style scoped>
.overview-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
}

/* 顶部统计卡片行 */
.top-row {
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 20rpx;
}

.info-box {
  flex: 1;
  min-width: 200rpx;
  background-color: #f3f7ff;
  margin: 10rpx;
  padding: 20rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.info-title {
  font-size: 28rpx;
  color: #666;
}

.info-value {
  font-size: 36rpx;
  color: #333;
  margin-top: 10rpx;
  font-weight: bold;
}

/* 年级课堂专注度图表 */
.chart-row {
  background-color: #fff;
  margin-bottom: 20rpx;
  padding: 20rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
}

.chart-header {
  margin-bottom: 10rpx;
}

.chart-title {
  font-size: 32rpx;
  font-weight: bold;
}

.charts-box {
  width: 100%;
  height: 300rpx; /* 确保图表容器有足够高度 */
}

/* 前10名班级表格 */
.table-row {
  background-color: #fff;
  padding: 20rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
}

.table-title {
  font-size: 32rpx;
  font-weight: bold;
  margin-bottom: 10rpx;
}

.table {
  width: 100%;
}

.table-header {
  display: flex;
  background-color: #f2f2f2;
  padding: 10rpx 0;
  margin-bottom: 5rpx;
}

.table-th {
  flex: 1;
  font-weight: bold;
  text-align: center;
}

.table-body {
  display: flex;
  padding: 8rpx 0;
  border-bottom: 1rpx solid #eee;
}

.table-td {
  flex: 1;
  text-align: center;
}
</style>