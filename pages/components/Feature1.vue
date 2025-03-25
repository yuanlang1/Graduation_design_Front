<template>
  <div class="data-detection-dashboard">
    <!-- 左右布局容器 -->
    <div class="main-container">
      <!-- 左侧：课堂信息 + 图表 + 统计信息 -->
      <div class="left-container">
        <!-- 课堂信息栏 -->
        <div class="class-info-bar">
          <div class="class-info-item">{{ courseName }}</div>
          <div class="class-info-item">{{ classTime }}</div>
          <div class="class-info-item">{{ className }}</div>
          <div class="class-info-item">{{ teacher }}</div>
        </div>
        <!-- 行为数据曲线图 -->
        <div class="chart-container">
          <div ref="chartRef" class="chart-box"></div>
        </div>
        <!-- 底部统计信息展示 -->
        <div class="bottom-container">
          <div class="info-item">
            <div class="label">举手次数</div>
            <div class="value font">{{ handCount }}</div>
          </div>
          <!-- 如果需要显示上课时长，可取消注释下面代码 -->
          <!-- <div class="info-item">
            <div class="label">上课时长</div>
            <div class="value font">{{ uploadDuration }}</div>
          </div> -->
          <div class="info-item">
            <div class="label">分数</div>
            <div class="value font">{{ score }}</div>
          </div>
        </div>
      </div>
      <!-- 右侧：视频区域（原始视频 + 检测后视频） -->
      <div class="right-container">
        <div class="video-box">
          <video
            ref="originalVideo"
            class="video-player"
            :src="originalVideoUrl"
            controls
            @loadedmetadata="onVideoLoaded"
            @timeupdate="onVideoTimeUpdate('original')"
            @play="onVideoPlay('original')"
            @pause="onVideoPause('original')"
          ></video>
        </div>
        <div class="video-box">
          <video
            ref="detectionVideo"
            class="video-player"
            :src="detectionVideoUrl"
            controls
            @timeupdate="onVideoTimeUpdate('detection')"
            @play="onVideoPlay('detection')"
            @pause="onVideoPause('detection')"
          ></video>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
export default {
  name: 'DataDetectionDashboard',
  data() {
    return {
      // 课程信息
      courseName: '数学',
      classTime: '2024/1/4 08:00:00',
      className: '高一一班',
      teacher: '张三',
      // 视频地址（默认值，可覆盖）
      originalVideoUrl: 'https://www.example.com/original.mp4',
      detectionVideoUrl: 'https://www.example.com/detection.mp4',

      // 图表及统计数据
      chartInstance: null,
      // actionData 中的 time 表示帧数（视频帧率为30，即1秒30帧）
      actionData: [
        { time: 0, dx: 0, dk: 0, tt: 0, zt: 0, js: 0, zl: 0, xt: 0 },
      ],
      // 默认动作类型，与 JSON 数据中的 key 保持一致
      actionTypes: ['dx', 'dk', 'tt', 'zt', 'js', 'zl', 'xt'],
      // 对应中文说明
      actionMap: {
        dx: "低头写字",
        dk: "低头看书",
        tt: "抬头听课",
        zt: "转头",
        js: "举手",
        zl: "站立",
        xt: "小组讨论"
      },
      currentTime: 0,
      handCount: 0,
      uploadDuration: '00:00:00',  // 显示上课时长（视频时长）
      score: 0,
      evaluationScore: 0,
      isSyncing: false,
      videoDuration: 0,  // 保存视频时长（秒）
      // 防抖定时器，用于限制鼠标移动事件频率
      mouseMoveTimeout: null,
    }
  },
  onLoad(options) {
    const eventChannel = this.getOpenerEventChannel()
    eventChannel.on('acceptDataFromOpenerPage', (data) => {
      this.courseName = data.courseName || this.courseName
      this.classTime = new Date(data.courseTime).toLocaleString()
      this.teacher = data.courseTeacher || this.teacher
      this.className = data.classTeacher || this.className
      this.originalVideoUrl = "http://127.0.0.1:8000/static/rawpic" + data.rawpic || this.originalVideoUrl
      this.detectionVideoUrl = "http://127.0.0.1:8000/static/resultpic" + data.resultpic || this.detectionVideoUrl
      this.score = data.class_socre * 100
      console.log("更新视频地址：", this.originalVideoUrl, this.detectionVideoUrl)
      if (data.jsonPath) {
        uni.request({
          url: "http://127.0.0.1:8000/Class_manage/get_json" + data.jsonPath + "/",
          method: 'GET',
          success: (res) => {
            if (res.statusCode === 200) {
              try {
                const jsonData = res.data
                // 将 JSON 中的帧数数据转换为 actionData
                this.actionData = jsonData.map(item => ({
                  time: item.frame,
                  ...item.action_count
                }))
                this.initChart()
              } catch (e) {
                console.error("JSON 数据解析错误：", e)
              }
            } else {
              console.error("加载 JSON 失败，状态码：", res.statusCode)
            }
          },
          fail: (err) => {
            console.error("请求 JSON 失败：", err)
          }
        })
      } else {
        this.initChart()
      }
    })
  },
  mounted() {
    if (!this.chartInstance) {
      this.initChart()
    }
    console.log("视频元素：", this.$refs.originalVideo)
  },
  methods: {
    // 格式化秒数为 HH:MM:SS 字符串
    formatSeconds(seconds) {
      const h = Math.floor(seconds / 3600)
      const m = Math.floor((seconds % 3600) / 60)
      const s = Math.floor(seconds % 60)
      return [h, m, s].map(v => v.toString().padStart(2, '0')).join(':')
    },
    // 视频元数据加载完成时，获取视频时长并更新显示
    onVideoLoaded() {
      const video = this.$refs.originalVideo;
      console.log("video.readyState:", video.readyState);
      console.log("video.duration:", video.duration);
      if (video.readyState === 4 && video.duration > 0) {
        this.videoDuration = video.duration;
        this.uploadDuration = this.formatSeconds(video.duration);
      } else {
        console.warn("视频时长无法获取，尝试延迟获取...");
        setTimeout(() => {
          const duration = this.$refs.originalVideo.duration;
          if (duration > 0) {
            this.videoDuration = duration;
            this.uploadDuration = this.formatSeconds(duration);
          }
        }, 1000);
      }
    },
    // 初始化 ECharts 图表
    initChart() {
      const chartDom = this.$refs.chartRef;
      if (!chartDom) return;
      
      // 如果已有实例，则销毁
      if (this.chartInstance) {
        this.chartInstance.dispose();
      }
      
      this.chartInstance = echarts.init(chartDom);
      
      // 解析上课起始时间（假设 classTime 为课堂开始时间）
      let classStart;
      if (this.classTime.includes("/") && this.classTime.includes(":")) {
        classStart = new Date(this.classTime);
      } else {
        classStart = new Date();
      }
      
      const vm = this;
      const seriesData = this.actionTypes.map(actionType => ({
        name: this.actionMap[actionType],
        type: 'line',
        showSymbol: false,
        data: this.actionData.map(item => [ new Date(classStart.getTime() + (item.time / 30) * 1000), item[actionType] ])
      }));
      
      const option = {
        title: { left: 'center' },
        tooltip: {
          trigger: 'axis',
          formatter: function(params) {
            const t = new Date(params[0].value[0]);
            const formatTime = (date) => {
              let hh = date.getHours().toString().padStart(2, '0');
              let mm = date.getMinutes().toString().padStart(2, '0');
              let ss = date.getSeconds().toString().padStart(2, '0');
              return `${hh}:${mm}:${ss}`;
            }
            if (params.length > 0) {
              const offsetSeconds = (t.getTime() - classStart.getTime()) / 1000;
              const idx = vm.findNearestIndex(offsetSeconds);
              if (idx !== -1) {
                vm.handCount = vm.actionData[idx].js;
              }
            }
            let tooltipStr = '时间：' + formatTime(t) + '\n';
            params.forEach(item => {
              tooltipStr += item.marker + item.seriesName + ': ' + item.value[1] + '\n';
            });
            return tooltipStr;
          }
        },
        legend: {
          data: this.actionTypes.map(key => this.actionMap[key]),
          top: 30
        },
        xAxis: {
          type: 'time',
          name: '上课时间',
          min: classStart,
          max: this.videoDuration ? new Date(classStart.getTime() + this.videoDuration * 1000) : null,
          axisLabel: {
            fontSize: 10,
            formatter: function(value) {
              const date = new Date(value);
              const hh = date.getHours().toString().padStart(2, '0');
              const mm = date.getMinutes().toString().padStart(2, '0');
              return `${hh}:${mm}`;
            }
          }
        },
        yAxis: { type: 'value', name: '动作数量' },
        series: seriesData,
        dataZoom: [
          {
            type: 'slider',
            show: true,
            xAxisIndex: 0,
          }
        ]
      };
      
      this.chartInstance.setOption(option);
      
      // 鼠标移出时恢复默认举手次数
      this.chartInstance.on('mouseout', () => {
        if (this.actionData && this.actionData.length > 0) {
          this.handCount = this.actionData[0].js;
        }
      });
      
      // 图表点击事件：点击数据点时同步视频跳转
      this.chartInstance.on('click', params => {
        if (params && params.value && params.value[0] != null) {
          const clickedDate = new Date(params.value[0]);
          const offsetSeconds = (clickedDate.getTime() - classStart.getTime()) / 1000;
          vm.currentTime = offsetSeconds;
          if (vm.$refs.originalVideo) {
            vm.$refs.originalVideo.currentTime = offsetSeconds;
          }
          if (vm.$refs.detectionVideo) {
            vm.$refs.detectionVideo.currentTime = offsetSeconds;
          }
          vm.chartInstance.dispatchAction({
            type: 'showTip',
            seriesIndex: 0,
            dataIndex: vm.findNearestIndex(offsetSeconds),
          });
        }
      });
      
      // 【新增】鼠标移动事件：当鼠标悬停时，同步视频跳转并显示该时间点的画面
      this.chartInstance.on('mousemove', params => {
        // 防抖处理：每次触发后延迟更新，避免频繁更新 currentTime
        if (this.mouseMoveTimeout) {
          clearTimeout(this.mouseMoveTimeout);
        }
        this.mouseMoveTimeout = setTimeout(() => {
          if (params && params.value && params.value[0] != null) {
            const hoveredDate = new Date(params.value[0]);
            const offsetSeconds = (hoveredDate.getTime() - classStart.getTime()) / 1000;
            // 更新视频 currentTime，确保视频框显示该时间点的画面
            if (vm.$refs.originalVideo) {
              vm.$refs.originalVideo.currentTime = offsetSeconds;
            }
            if (vm.$refs.detectionVideo) {
              vm.$refs.detectionVideo.currentTime = offsetSeconds;
            }
          }
        }, 100); // 延迟 100 毫秒
      });
    },
    // 视频播放同步：播放时同步另一视频
    onVideoPlay(whichVideo) {
      const otherVideo = whichVideo === 'original' ? this.$refs.detectionVideo : this.$refs.originalVideo;
      if (otherVideo && otherVideo.paused) {
        otherVideo.play();
      }
    },
    // 视频暂停同步：暂停时同步另一视频
    onVideoPause(whichVideo) {
      const otherVideo = whichVideo === 'original' ? this.$refs.detectionVideo : this.$refs.originalVideo;
      if (otherVideo && !otherVideo.paused) {
        otherVideo.pause();
      }
    },
    // 视频播放进度更新时，同步另一视频和图表提示
    onVideoTimeUpdate(whichVideo) {
      if (this.isSyncing) return;
      this.isSyncing = true;
      const currentVideo = whichVideo === 'original' ? this.$refs.originalVideo : this.$refs.detectionVideo;
      const otherVideo = whichVideo === 'original' ? this.$refs.detectionVideo : this.$refs.originalVideo;
      this.currentTime = currentVideo.currentTime;
      if (otherVideo) {
        otherVideo.currentTime = this.currentTime;
      }
      if (this.chartInstance) {
        const nearestIndex = this.findNearestIndex(this.currentTime);
        if (nearestIndex !== -1) {
          this.chartInstance.dispatchAction({
            type: 'showTip',
            seriesIndex: 0,
            dataIndex: nearestIndex,
          });
        }
      }
      this.isSyncing = false;
    },
    // 根据传入的秒数找到最近的 actionData 数据点索引
    findNearestIndex(time) {
      let minDiff = Infinity, index = -1;
      this.actionData.forEach((item, i) => {
        const diff = Math.abs(item.time / 30 - time);
        if (diff < minDiff) {
          minDiff = diff;
          index = i;
        }
      });
      return index;
    }
  }
}
</script>

<style scoped>
@font-face {
  font-family: 'XiangJiaoKuanMaoShuaLingGanTi-2';
  src: url('~@/static/fonts/XiangJiaoKuanMaoShuaLingGanTi-2.ttf');
}
.font {
  font-family: 'XiangJiaoKuanMaoShuaLingGanTi-2', sans-serif;
  font-size: 60rpx;
}
.data-detection-dashboard {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 16px;
  box-sizing: border-box;
  background-color: #f5f5f5;
}
.main-container {
  display: flex;
  flex: 1;
  gap: 10px;
}
.left-container {
  display: flex;
  flex-direction: column;
  flex: 2;
  gap: 8px;
}
.class-info-bar {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  font-size: 16px;
  color: #333;
  gap: 70px;
}
.class-info-item {
  font-family: 'XiangJiaoKuanMaoShuaLingGanTi-2', sans-serif;
  font-size: 40rpx;
  text-align: left;
}
.chart-container {
  flex: 1;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  min-height: 500px;
}
.chart-box {
  width: 100%;
  height: 100%;
  min-height: 500px;
  min-width: 600px;
}
.bottom-container {
  display: flex;
  gap: 16px;
  margin-top: 8px;
  background: transparent;
  border: none;
  box-shadow: none;
  padding: 0;
}
.info-item {
  flex: 1;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
.right-container {
  display: flex;
  flex-direction: column;
  flex: 2;
  gap: 16px;
}
.video-box {
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 8px;
}
.video-player {
  width: 100%;
  height: 402px;
  background: #000;
  border-radius: 4px;
}
</style>  