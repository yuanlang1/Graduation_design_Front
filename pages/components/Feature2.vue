<template>
  <view class="container">
    <!-- 顶部查询区域 -->
    <view class="search-bar">
      <view class="search-item">
        <text>课堂名称：</text>
        <picker
          mode="selector"
          :value="searchParamsClassNameIndex"
          :range="classNameOptions"
          @change="handleSearchClassNameChange"
        >
          <view class="picker-text">
		  {{ searchParams.className || '请选择课堂名称' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>课程名称：</text>
        <picker
          mode="selector"
          :value="searchParamsCourseNameIndex"
          :range="courseNameOptions"
          @change="handleSearchCourseNameChange"
        >
          <view class="picker-text">
		  {{ searchParams.courseName || '请选择课程名称' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>教师名称：</text>
        <input
          type="text"
          placeholder="请输入教师名称"
          v-model="searchParams.teacherName"
          @confirm="fetchClassrooms"
        />
      </view>
      <view class="search-item">
        <text>上课时间：</text>
        <picker
          mode="multiSelector"
          :value="[searchParamsYearIndex, searchParamsMonthIndex, searchParamsDayIndex, searchParamsHourIndex, searchParamsMinuteIndex, searchParamsSecondIndex]"
          :range="[yearOptions, monthOptions, dayOptions, hourOptions, minuteOptions, secondOptions]"
          @change="handleSearchDtimeChange"
          @columnchange="handleSearchDtimeColumnChange"
        >
          <view class="picker-text" style="width: 300rpx;">
			{{ searchParams.dtime || '请选择上课时间' }}
			<image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>上课地址：</text>
        <picker
          mode="selector"
          :value="searchParamsLocationIndex"
          :range="locationOptions"
          @change="handleSearchLocationChange"
        >
          <view class="picker-text">
		  {{ searchParams.location || '请选择上课地址' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>上课节数：</text>
        <picker
          mode="selector"
          :value="searchParamsStageIndex"
          :range="stageOptions"
          @change="handleSearchStageChange"
        >
          <view class="picker-text">
		  {{ searchParams.stage || '请选择上课节数' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <button type="primary" @click="fetchClassrooms" style="margin-left: 10rpx; margin-right: 20rpx;">查询</button>
      <button type="primary" @click="addClassroom" style="margin-left: 30rpx;">添加课堂</button>
    </view>

    <!-- 课程信息列表 -->
    <view class="table-container">
      <view class="table-header">
        <text class="table-th" style="flex: 1;">课程名称</text>
        <text class="table-th" style="flex: 1;">课堂名称</text>
        <text class="table-th" style="flex: 1;">老师</text>
        <text class="table-th" style="flex: 1;">上课时间</text>
        <text class="table-th" style="flex: 1;">上课节数</text>
        <text class="table-th" style="flex: 1;">上课地址</text>
        <text class="table-th" style="flex: 1;">原始媒体</text>
        <text class="table-th" style="flex: 1;">检测后媒体</text>
        <text class="table-th" style="flex: 1;">检测时间</text>
        <text class="table-th" style="flex: 1;">专注度分数</text>
        <text class="table-th" style="flex: 1;">操作</text>
      </view>
      <view class="table-body" v-for="(item, index) in classroomList" :key="index">
        <text class="table-td" style="flex: 1;">{{ item.course_name }}</text>
        <text class="table-td" style="flex: 1;">{{ item.class_name }}</text>
        <text class="table-td" style="flex: 1;">{{ item.teacher_name }}</text>
        <text class="table-td" style="flex: 1;">{{ formatDate(item.dtime) }}</text>
        <text class="table-td" style="flex: 1;">{{ item.stage }}</text>
        <text class="table-td" style="flex: 1;">{{ item.location }}</text>
        <view class="table-td" style="flex: 1;">
          <text v-if="item.rawpic" class="image-link" @tap="previewMedia(getMediaUrl(item.rawpic))">查看</text>
          <text v-else>无</text>
        </view>
        <view class="table-td" style="flex: 1;">
          <text v-if="item.resultpic" class="image-link" @tap="previewMedia(getMediaUrl(item.resultpic, 'resultpic'))">查看</text>
          <text v-else>无</text>
        </view>
        <text class="table-td" style="flex: 1;">{{ formatDate(item.detect_time) }}</text>
        <text class="table-td" style="flex: 1;">{{ item.estimate }}</text>
        <view class="table-td" style="flex: 1;">
          <button type="warn" size="mini" @click="deleteClassroom(item.id)">删除</button>
		  <button type="primary" size="mini" style="margin-left: 5rpx;" @click="jumpToPage(item)">跳转</button>
        </view>
      </view>
      <view v-if="classroomList.length === 0" class="no-data">暂无数据</view>
    </view>

    <!-- 添加/编辑课堂弹窗 -->
    <view class="modal" v-if="showModal">
      <view class="modal-content">
        <view class="modal-header">
          <text>{{ modalTitle }}</text>
          <text class="close-btn" @click="closeModal">×</text>
        </view>
        <view class="modal-body">
          <view class="form-item">
            <text>课堂视频：</text>
            <view v-if="courseForm.previewMedia" class="media-preview">
              <image
                v-if="courseForm.mediaType === 'image'"
                :src="courseForm.previewMedia"
                class="preview-image"
                mode="aspectFill"
                @tap="previewMedia"
              />
              <video
                v-else-if="courseForm.mediaType === 'video'"
                :src="courseForm.previewMedia"
                class="preview-video"
                controls
                @tap="previewMedia"
              ></video>
              <text class="remove-btn" @click="removeMedia">×</text>
            </view>
            <view v-else class="media-buttons">
              <button type="primary" size="mini" style="margin-left: 20rpx;" @click="chooseVideo">上传视频</button>
            </view>
          </view>
          <view class="form-item">
            <text>课堂名称：</text>
            <picker
              mode="selector"
              :value="courseFormClassNameIndex"
              :range="classNameOptions"
              @change="handleFormClassNameChange"
            >
              <view class="picker-text">
			  {{ courseForm.className || '请选择课堂名称' }}
			  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>课程名称：</text>
            <picker
              mode="selector"
              :value="courseFormCourseNameIndex"
              :range="courseNameOptions"
              @change="handleFormCourseNameChange"
            >
              <view class="picker-text">
			  {{ courseForm.courseName || '请选择课程名称' }}
			  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>教师名称：</text>
            <input
              type="text"
              v-model="courseForm.teacherName"
              placeholder="请输入教师名称"
            />
          </view>
          <view class="form-item">
            <text>上课时间：</text>
            <picker
              mode="multiSelector"
              :value="[courseFormYearIndex, courseFormMonthIndex, courseFormDayIndex, courseFormHourIndex, courseFormMinuteIndex, courseFormSecondIndex]"
              :range="[yearOptions, monthOptions, dayOptions, hourOptions, minuteOptions, secondOptions]"
              @change="handleFormDtimeChange"
              @columnchange="handleFormDtimeColumnChange"
            >
              <view class="picker-text" style="width: 300rpx;">
			  {{ formatDate(courseForm.dtime) || '请选择上课时间' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
        </view>
        <view class="modal-footer">
          <button type="primary" @click="submitClassroom">提交</button>
          <button type="default" @click="closeModal">取消</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      searchParams: {
        className: '',
        courseName: '',
        teacherName: '',
        dtime: '',
        location: '',
        stage: ''
      },
      searchParamsClassNameIndex: 0,
      searchParamsCourseNameIndex: 0,
      searchParamsTeacherNameIndex: 0,
      searchParamsYearIndex: 0,
      searchParamsMonthIndex: 0,
      searchParamsDayIndex: 0,
      searchParamsHourIndex: 0,
      searchParamsMinuteIndex: 0,
      searchParamsSecondIndex: 0,
      searchParamsLocationIndex: 0,
      searchParamsStageIndex: 0,
      classroomList: [],
      showModal: false,
      modalTitle: '添加课堂',
      courseForm: {
        id: null,
        imageUrl: '', // 后端返回的媒体 URL
        previewMedia: '', // 本地预览的媒体路径
        mediaType: '', // 媒体类型（'image' 或 'video'）
        className: '',
        courseName: '',
        teacherName: '',
        dtime: ''
      },
      courseFormClassNameIndex: 0,
      courseFormCourseNameIndex: 0,
      courseFormYearIndex: 0,
      courseFormMonthIndex: 0,
      courseFormDayIndex: 0,
      courseFormHourIndex: 0,
      courseFormMinuteIndex: 0,
      courseFormSecondIndex: 0,
      classNameOptions: this.generateClassNameOptions(),
      courseNameOptions: ['数学课', '语文课', '英语课', '物理课', '生物课', '化学课', '音乐课', '体育课', '历史课', '信息技术课'],
      locationOptions: ['一号楼', '二号楼', '三号楼', '体育馆', '逸夫楼'],
      stageOptions: Array.from({ length: 6 }, (_, i) => (i + 1).toString()),
      yearOptions: Array.from({ length: 10 }, (_, i) => (new Date().getFullYear() + i).toString()),
      monthOptions: Array.from({ length: 12 }, (_, i) => String(i + 1).padStart(2, '0')),
      dayOptions: this.generateDayOptions(new Date().getFullYear(), 1),
      hourOptions: Array.from({ length: 24 }, (_, i) => String(i).padStart(2, '0')),
      minuteOptions: Array.from({ length: 60 }, (_, i) => String(i).padStart(2, '0')),
      secondOptions: Array.from({ length: 60 }, (_, i) => String(i).padStart(2, '0')),
      teacherNameOptions: []
    }
  },
  onLoad() {
    this.fetchAllClassrooms()
    this.initializeTeacherNameOptions()
  },
  methods: {
    // 初始化教师姓名选项
    initializeTeacherNameOptions() {
      this.teacherNameOptions = ['张三', '李四', '王五', '赵六', '']
      this.teacherNameOptions = [...new Set(this.classroomList.map(item => item.teacher_name).concat(['']))]
    },

    // 生成课堂名称选项
    generateClassNameOptions() {
      const grades = ['一', '二', '三', '四', '五', '六']
      const classes = ['一', '二', '三', '四', '五', '六', '七', '八', '九', '十']
      const options = []
      grades.forEach(grade => {
        classes.forEach(classNum => {
          options.push(`${grade}年级${classNum}班`)
        })
      })
      return options
    },

    // 生成日期选项
    generateDayOptions(year, month) {
      const isLeapYear = (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0)
      const daysInMonth = new Date(year, month, 0).getDate()
      return Array.from({ length: daysInMonth }, (_, i) => String(i + 1).padStart(2, '0'))
    },

    // 动态调整日期选项
    updateDayOptions(year, month) {
      const yearNum = parseInt(year)
      const monthNum = parseInt(month)
      this.dayOptions = this.generateDayOptions(yearNum, monthNum)
      const maxDays = this.dayOptions.length
      if (this.searchParamsDayIndex >= maxDays) this.searchParamsDayIndex = 0
      if (this.courseFormDayIndex >= maxDays) this.courseFormDayIndex = 0
    },

    // 格式化时间
    formatDate(date) {
      if (!date) return '无'
      return new Date(date).toLocaleString()
    },

    // 获取媒体 URL
    getMediaUrl(path, type = 'rawpic') {
      if (!path) return '';
      const extension = path.split('.').pop().toLowerCase();
      const isImage = ['jpg', 'jpeg', 'png'].includes(extension);
      const isVideo = ['mp4', 'avi', 'mov'].includes(extension);
      if (!isImage && !isVideo) return ''; 
	  if(type === 'rawpic'){
	    console.log(`http://127.0.0.1:8000/static/${type}/${path.replace(/^\//, '')}`);
		return `http://127.0.0.1:8000/static/${type}/${path.replace(/^\//, '')}`;
	  }
	  else if(type === 'resultpic'){
	    console.log(`http://127.0.0.1:8000/static/${type}${path}`);
	    return `http://127.0.0.1:8000/static/${type}${path}`
	  }
    },

    // 预览媒体（图片或视频）
    previewMedia(url) {
      const extension = url.split('.').pop().toLowerCase();
      if (['jpg', 'jpeg', 'png'].includes(extension)) {
        uni.previewImage({
          urls: [url],
          current: url
        });
      } else if (['mp4', 'avi', 'mov'].includes(extension)) {
        uni.navigateTo({
          url: `/pages/videoPreview?src=${encodeURIComponent(url)}`
        });
      }
    },

    // 加载所有课堂
    async fetchAllClassrooms() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/Class_manage/classrooms',
          method: 'GET',
          data: {}
        })
        if (res.data.code === 200) {
          this.classroomList = res.data.data.map(item => ({
            id: item.id,
            course_name: item.course_name || '未知课程',
            class_name: item.class_name || '未知课堂',
            teacher_name: item.teacher_name || '未知教师',
            dtime: item.course_time || '',
            stage: item.stage || 1,
            location: item.location || '',
            detect_time: item.detect_time || '',
            estimate: item.estimate || '0.0',
            rawpic: item.rawpic || '',
            resultpic: item.resultpic || '',
			result_json: item.result_json
          }))
		  console.log(classroomList)
          this.initializeTeacherNameOptions()
        } else {
          uni.showToast({ title: '加载课堂失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 查询课堂
    async fetchClassrooms() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/Class_manage/classrooms',
          method: 'GET',
          data: {
            className: this.searchParams.className,
            courseName: this.searchParams.courseName,
            teacherName: this.searchParams.teacherName,
            dtime: this.searchParams.dtime ? `${this.searchParams.dtime}` : '',
            location: this.searchParams.location,
            stage: this.searchParams.stage
          }
        })
        if (res.data.code === 200) {
          this.classroomList = res.data.data.map(item => ({
            id: item.id,
            course_name: item.course_name || '未知课程',
            class_name: item.class_name || '未知课堂',
            teacher_name: item.teacher_name || '未知教师',
            dtime: item.course_time || '',
            stage: item.stage || 1,
            location: item.location || '',
            detect_time: item.detect_time || '',
            estimate: item.estimate || '0.0',
            rawpic: item.rawpic || '',
            resultpic: item.resultpic || '',
			result_json: item.result_json || ''
          }))
        } else {
          uni.showToast({ title: '查询课堂失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 处理搜索栏课堂名称选择变化
    handleSearchClassNameChange(e) {
      this.searchParamsClassNameIndex = e.detail.value
      this.searchParams.className = this.classNameOptions[e.detail.value]
    },

    // 处理搜索栏课程名称选择变化
    handleSearchCourseNameChange(e) {
      this.searchParamsCourseNameIndex = e.detail.value
      this.searchParams.courseName = this.courseNameOptions[e.detail.value]
    },

    // 处理搜索栏教师姓名选择变化
    handleSearchTeacherNameChange(e) {
      this.searchParamsTeacherNameIndex = e.detail.value
      this.searchParams.teacherName = this.teacherNameOptions[e.detail.value] || this.searchParams.teacherName
    },

    // 处理搜索栏上课时间选择变化
    handleSearchDtimeChange(e) {
      const values = e.detail.value
      this.searchParamsYearIndex = values[0]
      this.searchParamsMonthIndex = values[1]
      this.searchParamsDayIndex = values[2]
      this.searchParamsHourIndex = values[3]
      this.searchParamsMinuteIndex = values[4]
      this.searchParamsSecondIndex = values[5]
      this.searchParams.dtime = `${this.yearOptions[values[0]]}-${this.monthOptions[values[1]]}-${this.dayOptions[values[2]]} ${this.hourOptions[values[3]]}:${this.minuteOptions[values[4]]}:${this.secondOptions[values[5]]}`
    },

    // 处理搜索栏上课时间列变化
    handleSearchDtimeColumnChange(e) {
      const column = e.detail.column
      const value = e.detail.value
      if (column === 0) {
        this.searchParamsYearIndex = value
        this.updateDayOptions(this.yearOptions[value], this.monthOptions[this.searchParamsMonthIndex])
      } else if (column === 1) {
        this.searchParamsMonthIndex = value
        this.updateDayOptions(this.yearOptions[this.searchParamsYearIndex], this.monthOptions[value])
      }
    },

    // 处理搜索栏上课地址选择变化
    handleSearchLocationChange(e) {
      this.searchParamsLocationIndex = e.detail.value
      this.searchParams.location = this.locationOptions[e.detail.value]
    },

    // 处理搜索栏上课节数选择变化
    handleSearchStageChange(e) {
      this.searchParamsStageIndex = e.detail.value
      this.searchParams.stage = this.stageOptions[e.detail.value]
    },

    // 选择并上传图片
    async chooseImage() {
      try {
        console.log('开始选择图片...');
        const res = await uni.chooseImage({
          count: 1,
          sizeType: ['original', 'compressed'],
          sourceType: ['album', 'camera']
        });
        const tempFilePath = res.tempFilePaths[0];
        this.courseForm.previewMedia = tempFilePath;
        this.courseForm.mediaType = 'image';
        console.log('图片选择成功，路径:', tempFilePath);

        const uploadRes = await uni.uploadFile({
          url: 'http://127.0.0.1:8000/Class_manage/Upload_image/',
          filePath: tempFilePath,
          name: 'file',
          formData: { type: 'course', mediaType: 'image' },
          timeout: 30000
        });
        const uploadData = JSON.parse(uploadRes.data);
        if (uploadData.code === 200) {
          this.courseForm.imageUrl = uploadData.data.url;
          uni.showToast({ title: '上传图片成功', icon: 'success' });
        } else {
          uni.showToast({ title: `上传失败: ${uploadData.msg || '未知错误'}`, icon: 'none' });
          this.courseForm.previewMedia = '';
          this.courseForm.mediaType = '';
        }
      } catch (error) {
        console.error('上传图片出错:', error);
        uni.showToast({ title: '上传图片出错，请重试', icon: 'none' });
        this.courseForm.previewMedia = '';
        this.courseForm.mediaType = '';
      }
    },

    // 选择并上传视频
    async chooseVideo() {
      try {
        console.log('开始选择视频...');
        const res = await uni.chooseVideo({
          count: 1,
          sourceType: ['album', 'camera'],
          maxDuration: 60,
          camera: 'back'
        });
        const tempFilePath = res.tempFilePath;
        this.courseForm.previewMedia = tempFilePath;
        this.courseForm.mediaType = 'video';
        console.log('视频选择成功，路径:', tempFilePath);

        const uploadRes = await uni.uploadFile({
          url: 'http://127.0.0.1:8000/Class_manage/Upload_image/',
          filePath: tempFilePath,
          name: 'file',
          formData: { type: 'course', mediaType: 'video' },
          timeout: 30000
        });
        const uploadData = JSON.parse(uploadRes.data);
        if (uploadData.code === 200) {
          this.courseForm.imageUrl = uploadData.data.url;
          uni.showToast({ title: '上传视频成功', icon: 'success' });
        } else {
          uni.showToast({ title: `上传失败: ${uploadData.msg || '未知错误'}`, icon: 'none' });
          this.courseForm.previewMedia = '';
          this.courseForm.mediaType = '';
        }
      } catch (error) {
        console.error('上传视频出错:', error);
        uni.showToast({ title: '上传视频出错，请重试', icon: 'none' });
        this.courseForm.previewMedia = '';
        this.courseForm.mediaType = '';
      }
    },

    // 移除媒体
    removeMedia() {
      this.courseForm.previewMedia = '';
      this.courseForm.mediaType = '';
      this.courseForm.imageUrl = '';
    },

    // 打开添加课堂弹窗
    addClassroom() {
      this.modalTitle = '添加课堂'
      this.courseForm = {
        id: null,
        imageUrl: '',
        previewMedia: '',
        mediaType: '',
        className: '',
        courseName: '',
        teacherName: '',
        dtime: ''
      }
      this.courseFormClassNameIndex = 0
      this.courseFormCourseNameIndex = 0
      this.courseFormYearIndex = 0
      this.courseFormMonthIndex = 0
      this.courseFormDayIndex = 0
      this.courseFormHourIndex = 0
      this.courseFormMinuteIndex = 0
      this.courseFormSecondIndex = 0
      this.showModal = true
    },

    // 编辑课堂
    editClassroom(item) {
      this.modalTitle = '编辑课堂'
      this.courseForm = {
        id: item.id,
        imageUrl: item.rawpic || '',
        previewMedia: '',
        mediaType: item.rawpic ? (this.getMediaType(item.rawpic) === 'image' ? 'image' : 'video') : '',
        className: item.class_name,
        courseName: item.course_name,
        teacherName: item.teacher_name,
        dtime: item.dtime || ''
      }
      this.courseFormClassNameIndex = this.classNameOptions.indexOf(item.class_name)
      this.courseFormCourseNameIndex = this.courseNameOptions.indexOf(item.course_name)
      this.searchParamsTeacherNameIndex = this.teacherNameOptions.indexOf(item.teacher_name) >= 0 ? this.teacherNameOptions.indexOf(item.teacher_name) : 0
      const [datePart, timePart] = (item.dtime || '').split(' ')
      if (datePart && timePart) {
        const [year, month, day] = datePart.split('-')
        const [hour, minute, second] = timePart.split(':')
        this.courseFormYearIndex = this.yearOptions.indexOf(year)
        this.courseFormMonthIndex = this.monthOptions.indexOf(month)
        this.courseFormDayIndex = this.dayOptions.indexOf(day)
        this.courseFormHourIndex = this.hourOptions.indexOf(hour)
        this.courseFormMinuteIndex = this.minuteOptions.indexOf(minute)
        this.courseFormSecondIndex = this.secondOptions.indexOf(second)
        this.updateDayOptions(year, month)
      } else {
        this.courseFormYearIndex = 0
        this.courseFormMonthIndex = 0
        this.courseFormDayIndex = 0
        this.courseFormHourIndex = 0
        this.courseFormMinuteIndex = 0
        this.courseFormSecondIndex = 0
      }
      this.showModal = true
    },

    // 删除课堂
    async deleteClassroom(id) {
      uni.showModal({
        title: '提示',
        content: '确定删除此课堂吗？',
        success: async (res) => {
          if (res.confirm) {
            try {
              const classroom = this.classroomList.find(item => item.id === id)
              if (!classroom) {
                uni.showToast({ title: '课堂不存在', icon: 'none' })
                return
              }
              const deleteRes = await uni.request({
                url: `http://127.0.0.1:8000/Class_manage/Delete_classrooms/${id}/`,
                method: 'DELETE',
                data: {
				  dtime: classroom.detect_time
                }
              })
              if (deleteRes.data.code === 200) {
                uni.showToast({ title: '删除成功', icon: 'success' })
                this.fetchAllClassrooms()
              } else {
                uni.showToast({ title: '删除失败', icon: 'none' })
              }
            } catch (error) {
              uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
            }
          }
        }
      })
    },

    // 处理表单课堂名称选择变化
    handleFormClassNameChange(e) {
      this.courseFormClassNameIndex = e.detail.value
      this.courseForm.className = this.classNameOptions[e.detail.value]
    },

    // 处理表单课程名称选择变化
    handleFormCourseNameChange(e) {
      this.courseFormCourseNameIndex = e.detail.value
      this.courseForm.courseName = this.courseNameOptions[e.detail.value]
    },

    // 处理表单上课时间选择变化
    handleFormDtimeChange(e) {
      const values = e.detail.value
      this.courseFormYearIndex = values[0]
      this.courseFormMonthIndex = values[1]
      this.courseFormDayIndex = values[2]
      this.courseFormHourIndex = values[3]
      this.courseFormMinuteIndex = values[4]
      this.courseFormSecondIndex = values[5]
      this.courseForm.dtime = `${this.yearOptions[values[0]]}-${this.monthOptions[values[1]]}-${this.dayOptions[values[2]]} ${this.hourOptions[values[3]]}:${this.minuteOptions[values[4]]}:${this.secondOptions[values[5]]}`
    },

    // 处理表单上课时间列变化
    handleFormDtimeColumnChange(e) {
      const column = e.detail.column
      const value = e.detail.value
      if (column === 0) {
        this.courseFormYearIndex = value
        this.updateDayOptions(this.yearOptions[value], this.monthOptions[this.courseFormMonthIndex])
      } else if (column === 1) {
        this.courseFormMonthIndex = value
        this.updateDayOptions(this.yearOptions[this.courseFormYearIndex], this.monthOptions[value])
      }
    },

    // 获取媒体类型
    getMediaType(url) {
      const extension = url.split('.').pop().toLowerCase();
      if (['jpg', 'jpeg', 'png'].includes(extension)) {
        return 'image';
      } else if (['mp4', 'avi', 'mov'].includes(extension)) {
        return 'video';
      }
      return 'unknown'; // 未知类型
    },
	
	 // 轮询任务状态
	  pollTaskStatus(taskId) {
		const interval = setInterval(async () => {
		  try {
			const statusRes = await uni.request({
			  url: `http://127.0.0.1:8000/Class_manage/task_status/${taskId}/`,
			  method: 'GET'
			});
			if (statusRes.data.state === 'SUCCESS') {
			  // 任务执行成功后，可更新课堂列表或提示用户检测完成
			  uni.showToast({ title: '检测完成', icon: 'success' });
			  // 此处可刷新课堂数据，或者直接更新该课堂的状态
			  this.fetchAllClassrooms();
			  clearInterval(interval);
			} else if (statusRes.data.state === 'FAILURE') {
			  uni.showToast({ title: '检测任务失败', icon: 'none' });
			  clearInterval(interval);
			}
			// 如果状态是 PENDING 或 STARTED，则继续轮询
		  } catch (error) {
			console.error('轮询任务状态错误:', error);
			clearInterval(interval);
		  }
		}, 3000); // 每 3 秒查询一次
	},

    // 提交课堂
    async submitClassroom() {
      try {
        const method = this.courseForm.id ? 'PUT' : 'POST'
        const url = this.courseForm.id
          ? `http://127.0.0.1:8000/Class_manage/classrooms/${this.courseForm.id}/`
          : 'http://127.0.0.1:8000/Class_manage/Add_classrooms/'

        if (this.courseForm.dtime && !/^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$/.test(this.courseForm.dtime)) {
          uni.showToast({ title: '时间格式错误，请输入 YYYY-MM-DD HH:MM:SS', icon: 'none' })
          return
        }

        const res = await uni.request({
          url: url,
          method: method,
          data: {
            className: this.courseForm.className,
            courseName: this.courseForm.courseName,
            teacherName: this.courseForm.teacherName,
            dtime: this.courseForm.dtime,
            rawpic: this.courseForm.imageUrl
          }
        })
        if (res.data.code === 200) {
         const taskId = res.data.task_id;
           uni.showToast({ title: '任务已提交', icon: 'success' });
		   console.log("id:", res.data.task_id)
           // 开启轮询查询任务状态
           this.pollTaskStatus(taskId);
        }
		else {
		   uni.showToast({ title: '添加失败', icon: 'none' });
		}
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },
	jumpToPage(item) {
		const dataToSend = {
			courseName: item.course_name,    
			courseTime: item.dtime,            
			courseTeacher: item.teacher_name, 
			classTeacher: item.class_name,
			class_socre: item.estimate,
			rawpic: item.rawpic,               
			resultpic: item.resultpic,         
			jsonPath: item.result_json          
		}
		console.log(dataToSend)
		uni.navigateTo({
			url: '/pages/components/Feature1', 
			success: (res) => {
			  res.eventChannel.emit('acceptDataFromOpenerPage', dataToSend);
			}
		})
	},

    // 关闭弹窗
    closeModal() {
      this.showModal = false
      this.courseForm = {
        id: null,
        imageUrl: '',
        previewMedia: '',
        mediaType: '',
        className: '',
        courseName: '',
        teacherName: '',
        dtime: ''
      }
      this.courseFormClassNameIndex = 0
      this.courseFormCourseNameIndex = 0
      this.courseFormYearIndex = 0
      this.courseFormMonthIndex = 0
      this.courseFormDayIndex = 0
      this.courseFormHourIndex = 0
      this.courseFormMinuteIndex = 0
      this.courseFormSecondIndex = 0
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  padding: 20rpx;
  background-color: #f7f8fa;
}

.search-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 10rpx;
  margin-bottom: 20rpx;
  background-color: #fff;
  padding: 20rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
  align-items: center;
}

.search-item {
  display: flex;
  align-items: center;
  margin-right: 10rpx;
}

.search-item text {
  margin-right: 10rpx;
  font-size: 28rpx;
  color: #333;
}

.search-item input {
  width: 250rpx;
  height: 40rpx;
  padding: 0 10rpx;
  border: 1rpx solid #eee;
  border-radius: 4rpx;
  font-size: 28rpx;
}

button {
  height: 40rpx;
  line-height: 40rpx;
  font-size: 28rpx;
  margin-top: 10rpx;
}

.table-container {
  flex: 1;
  background-color: #fff;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.table-header, .table-body {
  display: flex;
  padding: 10rpx 20rpx;
  border-bottom: 1rpx solid #eee;
}

.table-th, .table-td {
  flex: 1;
  font-size: 28rpx;
  color: #333;
  text-align: center;
}

.table-th {
  font-weight: bold;
  background-color: #f2f2f2;
}

.table-body {
  background-color: #fff;
}

.image-link {
  color: #007bff;
  text-decoration: underline;
  cursor: pointer;
}

.no-data {
  text-align: center;
  padding: 20rpx;
  color: #999;
  font-size: 28rpx;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  width: 100%;
  max-width: 900rpx;
  max-height: 1050rpx;
  background-color: #fff;
  border-radius: 8rpx;
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  padding: 20rpx;
  border-bottom: 1rpx solid #eee;
  font-size: 32rpx;
  font-weight: bold;
}

.close-btn {
  font-size: 36rpx;
  cursor: pointer;
}

.modal-body {
  padding: 20rpx;
}

.form-item {
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
}

.form-item text {
  width: 140rpx;
  font-size: 28rpx;
  color: #333;
}

.form-item input {
  flex: 1;
  height: 40rpx;
  padding: 0 10rpx;
  border: 1rpx solid #eee;
  border-radius: 4rpx;
  font-size: 28rpx;
  width: 300rpx;
}

.picker-text {
   position: relative; /* 设置为相对定位 */
   width: 240rpx;
   height: 40rpx;
   line-height: 40rpx;
   padding: 0 30rpx 0 10rpx; /* 右侧留出足够空间放置图标 */
   border: 1rpx solid #eee;
   border-radius: 4rpx;
   font-size: 28rpx;
   color: #333;
   background-color: #fff;
   cursor: pointer;
}

.media-preview {
  position: relative;
  margin-left: 20rpx;
}

.preview-image {
  width: 670rpx;
  height: 400rpx;
  border-radius: 4rpx;
}

.preview-video {
  width: 670rpx;
  height: 400rpx;
  border-radius: 4rpx;
}

.remove-btn {
  position: absolute;
  top: 10rpx;
  right: 10rpx;
  width: 40rpx;
  height: 40rpx;
  background-color: rgba(0, 0, 0, 0.5);
  color: #fff;
  text-align: center;
  line-height: 40rpx;
  border-radius: 50%;
  font-size: 28rpx;
  cursor: pointer;
}

.media-buttons {
  display: flex;
  align-items: center;
  margin-left: 20rpx;
}

.media-buttons button {
  height: 40rpx;
  line-height: 40rpx;
  font-size: 28rpx;
  margin-right: 10rpx;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  padding: 20rpx;
  gap: 20rpx;
  border-top: 1rpx solid #eee;
}

.modal-footer button {
  width: 120rpx;
  height: 40rpx;
  line-height: 40rpx;
  font-size: 28rpx;
}
/* 新增下拉标识样式 */
.picker-arrow {
  position: absolute;
  right: 10rpx; /* 图标距离容器右侧10rpx */
  top: 50%;
  transform: translateY(-50%);
  width: 20rpx;
  height: 20rpx;
}
.picker-arrow2 {
  position: absolute;
  right: 10rpx; /* 图标距离容器右侧10rpx */
  top: 50%;
  transform: translateY(-50%);
  width: 20rpx;
  height: 20rpx;
}
</style>