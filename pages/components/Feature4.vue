<template>
  <view class="container">
    <!-- 顶部查询区域 -->
    <view class="search-bar">
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
        <text>教师姓名：</text>
        <input
          type="text"
          placeholder="请输入教师姓名"
          v-model="searchParams.teacherName"
          @confirm="fetchCourses"
        />
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
        <text>上课时间：</text>
        <picker
          mode="multiSelector"
          :value="[searchParamsYearIndex, searchParamsMonthIndex, searchParamsDayIndex, searchParamsHourIndex, searchParamsMinuteIndex, searchParamsSecondIndex]"
          :range="[yearOptions, monthOptions, dayOptions, hourOptions, minuteOptions, secondOptions]"
          @change="handleSearchDtimeChange"
          @columnchange="handleSearchDtimeColumnChange"
        >
          <view class="picker-text" style="width: 310rpx;">
		  {{ searchParams.dtime || '请选择上课时间' }}
		  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <button type="primary" @click="fetchCourses" style="margin-left: 10rpx; margin-right: 20rpx;">查询</button>
      <button type="primary" @click="addCourse" style="margin-left: 30rpx;">添加课程</button>
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
        <text class="table-th" style="flex: 1;">操作</text>
      </view>
      <view class="table-body" v-for="(item, index) in courseList" :key="index">
        <text class="table-td" style="flex: 1;">{{ item.course_name }}</text>
        <text class="table-td" style="flex: 1;">{{ item.class_name }}</text>
        <text class="table-td" style="flex: 1;">{{ item.teacher_name }}</text>
        <text class="table-td" style="flex: 1;">{{ formatDate(item.dtime) }}</text>
        <text class="table-td" style="flex: 1;">{{ item.stage }}</text>
        <text class="table-td" style="flex: 1;">{{ item.location }}</text>
        <view class="table-td" style="flex: 1;">
          <button type="primary" size="mini" style="background-color: #45a049; color: aliceblue;" @click="editCourse(item)">修改</button>
          <button type="warn" size="mini" @click="deleteCourse(item.id)">删除</button>
        </view>
      </view>
      <view v-if="courseList.length === 0" class="no-data">暂无数据</view>
    </view>

    <!-- 添加/编辑课程弹窗 -->
    <view class="modal" v-if="showModal">
      <view class="modal-content">
        <view class="modal-header">
          <text>{{ modalTitle }}</text>
          <text class="close-btn" @click="closeModal">×</text>
        </view>
        <view class="modal-body">
          <view class="form-item">
            <text>课堂名称：</text>
            <picker
              mode="selector"
              :value="courseFormClassNameIndex"
              :range="classNameOptions"
              @change="handleFormClassNameChange"
            >
              <view class="picker-text" style="width: 250rpx;">
			  {{ courseForm.className || '请选择课堂名称' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
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
              <view class="picker-text" style="width: 250rpx;">
			  {{ courseForm.courseName || '请选择课程名称' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>教师姓名：</text>
            <input
              type="text"
              v-model="courseForm.teacherName"
              placeholder="请输入教师姓名"
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
              <view class="picker-text" style="width: 250rpx;">
			  {{ formatDate(courseForm.dtime) || '请选择上课时间' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>上课地址：</text>
            <picker
              mode="selector"
              :value="courseFormLocationIndex"
              :range="locationOptions"
              @change="handleFormLocationChange"
            >
              <view class="picker-text" style="width: 250rpx;">
			  {{ courseForm.location || '请选择上课地址' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>上课节数：</text>
            <picker
              mode="selector"
              :value="courseFormStageIndex"
              :range="stageOptions"
              @change="handleFormStageChange"
            >
              <view class="picker-text" style="width: 250rpx;">
			  {{ courseForm.stage || '请选择上课节数' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
        </view>
        <view class="modal-footer">
          <button type="primary" @click="submitCourse">提交</button>
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
        courseName: '',
        className: '',
        teacherName: '',
        stage: '', 
        location: '', 
        dtime: ''
      },
      searchParamsCourseNameIndex: 0, // 搜索栏课程名称的索引
      searchParamsClassNameIndex: 0, // 搜索栏课堂名称的索引
      searchParamsTeacherNameIndex: 0, // 搜索栏教师姓名的索引
      searchParamsStageIndex: 0, // 搜索栏上课节数的索引
      searchParamsLocationIndex: 0, // 搜索栏上课地址的索引
      searchParamsYearIndex: 0, // 搜索栏年份索引
      searchParamsMonthIndex: 0, // 搜索栏月份索引
      searchParamsDayIndex: 0, // 搜索栏日期索引
      searchParamsHourIndex: 0, // 搜索栏小时索引
      searchParamsMinuteIndex: 0, // 搜索栏分钟索引
      searchParamsSecondIndex: 0, // 搜索栏秒钟索引
      courseList: [],
      showModal: false,
      modalTitle: '添加课程',
      courseForm: {
        id: null,
        className: '',
        courseName: '',
        teacherName: '',
        dtime: '', 
        location: '', 
        stage: ''
      },
      courseFormClassNameIndex: 0, // 表单课堂名称的索引
      courseFormCourseNameIndex: 0, // 表单课程名称的索引
      courseFormYearIndex: 0, // 表单年份索引
      courseFormMonthIndex: 0, // 表单月份索引
      courseFormDayIndex: 0, // 表单日期索引
      courseFormHourIndex: 0, // 表单小时索引
      courseFormMinuteIndex: 0, // 表单分钟索引
      courseFormSecondIndex: 0, // 表单秒钟索引
      courseFormLocationIndex: 0, // 表单上课地址的索引
      courseFormStageIndex: 0, // 表单上课节数的索引
      classNameOptions: this.generateClassNameOptions(), // 课堂名称选项（一到六年级，一到十班）
      courseNameOptions: ['数学课', '语文课', '英语课', '物理课', '生物课', '化学课', '音乐课', '体育课', '历史课', '信息技术课'], // 课程名称选项
      locationOptions: ['一号楼', '二号楼', '三号楼', '体育馆', '逸夫楼'], // 上课地址选项
      stageOptions: Array.from({ length: 6 }, (_, i) => (i + 1).toString()), // 上课节数选项（1-6节）
      yearOptions: Array.from({ length: 10 }, (_, i) => (new Date().getFullYear() + i).toString()), // 年份选项（当前年份到未来10年）
      monthOptions: Array.from({ length: 12 }, (_, i) => String(i + 1).padStart(2, '0')), // 月份选项（01-12）
      dayOptions: this.generateDayOptions(new Date().getFullYear(), 1), // 日期选项（动态生成，初始为当前年1月）
      hourOptions: Array.from({ length: 24 }, (_, i) => String(i).padStart(2, '0')), // 小时选项（00-23）
      minuteOptions: Array.from({ length: 60 }, (_, i) => String(i).padStart(2, '0')), // 分钟选项（00-59）
      secondOptions: Array.from({ length: 60 }, (_, i) => String(i).padStart(2, '0')), // 秒钟选项（00-59）
      teacherNameOptions: [] // 教师姓名选项（动态从后端获取或初始化为常见值）
    }
  },
  onLoad() {
    // 进入页面时直接加载所有课程，并初始化教师姓名选项
    this.fetchAllCourses()
    this.initializeTeacherNameOptions()
  },
  methods: {
    // 初始化教师姓名选项（假设从后端获取或使用默认值）
    initializeTeacherNameOptions() {
      // 模拟从后端获取教师姓名列表（实际应通过 API 获取）
      this.teacherNameOptions = ['张三', '李四', '王五', '赵六', ''] // 示例值，可扩展
      // 或者通过 fetchAllCourses 获取后设置
      this.teacherNameOptions = [...new Set(this.courseList.map(item => item.teacher_name).concat(['']))]
    },

    // 生成课堂名称选项（一到六年级，一到十班）
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

    // 生成日期选项（根据闰年规则计算有效天数）
    generateDayOptions(year, month) {
      const isLeapYear = (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0)
      const daysInMonth = new Date(year, month, 0).getDate()
      return Array.from({ length: daysInMonth }, (_, i) => String(i + 1).padStart(2, '0'))
    },

    // 动态调整日期选项（根据年月计算有效天数）
    updateDayOptions(year, month) {
      const yearNum = parseInt(year)
      const monthNum = parseInt(month)
      this.dayOptions = this.generateDayOptions(yearNum, monthNum)
      // 重置日期索引，如果超出范围
      const maxDays = this.dayOptions.length
      if (this.searchParamsDayIndex >= maxDays) {
        this.searchParamsDayIndex = 0
      }
      if (this.courseFormDayIndex >= maxDays) {
        this.courseFormDayIndex = 0
      }
    },

    // 格式化时间
    formatDate(date) {
      if (!date) return '无'
      return new Date(date).toLocaleString()
    },

    // 加载所有课程（无条件）
    async fetchAllCourses() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/Course_manage/courses',
          method: 'GET',
          data: {} // 显式传递空对象，确保无条件查询
        })
        if (res.data.code === 200) {
          this.courseList = res.data.data.map(item => ({
            id: item.id,
            course_name: item.course_name || '未知课程',
            class_name: item.class_name || '未知课堂',
            teacher_name: item.teacher_name || '未知教师',
            dtime: item.dtime || '',
            location: item.location || '',
            stage: item.stage || 1
          }))
          // 更新教师姓名选项
          this.teacherNameOptions = [...new Set(this.courseList.map(item => item.teacher_name).concat(['']))]
        } else {
          uni.showToast({ title: '加载课程失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 查询课程（带条件）
    async fetchCourses() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/Course_manage/courses',
          method: 'GET',
          data: {
            course_name: this.searchParams.courseName,
            class_name: this.searchParams.className,
            teacher_name: this.searchParams.teacherName,
            stage: this.searchParams.stage,
            location: this.searchParams.location,
            dtime: this.searchParams.dtime ? `${this.searchParams.dtime}` : '' // 确保格式为 YYYY-MM-DD HH:MM:SS
          }
        })
        if (res.data.code === 200) {
          this.courseList = res.data.data.map(item => ({
            id: item.id,
            course_name: item.course_name || '未知课程',
            class_name: item.class_name || '未知课堂',
            teacher_name: item.teacher_name || '未知教师',
            dtime: item.dtime || '',
            location: item.location || '',
            stage: item.stage || 1
          }))
        } else {
          uni.showToast({ title: '查询课程失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 处理搜索栏课程名称选择变化
    handleSearchCourseNameChange(e) {
      this.searchParamsCourseNameIndex = e.detail.value
      this.searchParams.courseName = this.courseNameOptions[e.detail.value]
    },

    // 处理搜索栏课堂名称选择变化
    handleSearchClassNameChange(e) {
      this.searchParamsClassNameIndex = e.detail.value
      this.searchParams.className = this.classNameOptions[e.detail.value]
    },

    // 处理搜索栏教师姓名选择变化
    handleSearchTeacherNameChange(e) {
      this.searchParamsTeacherNameIndex = e.detail.value
      this.searchParams.teacherName = this.teacherNameOptions[e.detail.value]
    },

    // 处理搜索栏上课节数选择变化
    handleSearchStageChange(e) {
      this.searchParamsStageIndex = e.detail.value
      this.searchParams.stage = this.stageOptions[e.detail.value]
    },

    // 处理搜索栏上课地址选择变化
    handleSearchLocationChange(e) {
      this.searchParamsLocationIndex = e.detail.value
      this.searchParams.location = this.locationOptions[e.detail.value]
    },

    // 处理搜索栏上课时间选择变化（6列）
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

    // 处理搜索栏上课时间列变化（动态调整日期）
    handleSearchDtimeColumnChange(e) {
      const column = e.detail.column
      const value = e.detail.value
      if (column === 0) { // 年份变化
        this.searchParamsYearIndex = value
        this.updateDayOptions(this.yearOptions[value], this.monthOptions[this.searchParamsMonthIndex])
      } else if (column === 1) { // 月份变化
        this.searchParamsMonthIndex = value
        this.updateDayOptions(this.yearOptions[this.searchParamsYearIndex], this.monthOptions[value])
      }
    },

    // 打开添加课程弹窗
    addCourse() {
      this.modalTitle = '添加课程'
      this.courseForm = {
        id: null,
        className: '',
        courseName: '',
        teacherName: '',
        dtime: '', 
        location: '', 
        stage: ''
      }
      this.courseFormClassNameIndex = 0 // 重置课堂名称索引
      this.courseFormCourseNameIndex = 0 // 重置课程名称索引
      this.courseFormYearIndex = 0 // 重置年份索引
      this.courseFormMonthIndex = 0 // 重置月份索引
      this.courseFormDayIndex = 0 // 重置日期索引
      this.courseFormHourIndex = 0 // 重置小时索引
      this.courseFormMinuteIndex = 0 // 重置分钟索引
      this.courseFormSecondIndex = 0 // 重置秒钟索引
      this.courseFormLocationIndex = 0 // 重置上课地址索引
      this.courseFormStageIndex = 0 // 重置上课节数索引
      this.showModal = true
    },

    // 编辑课程
    editCourse(item) {
      this.modalTitle = '编辑课程'
      this.courseForm = {
        id: item.id,
        className: item.class_name,
        courseName: item.course_name,
        teacherName: item.teacher_name,
        dtime: item.dtime || '', 
        location: item.location || '', 
        stage: item.stage || ''
      }
      // 设置课堂名称索引
      this.courseFormClassNameIndex = this.classNameOptions.indexOf(item.class_name)
      // 设置课程名称索引
      this.courseFormCourseNameIndex = this.courseNameOptions.indexOf(item.course_name)
      // 设置教师姓名索引
      this.searchParamsTeacherNameIndex = this.teacherNameOptions.indexOf(item.teacher_name)
      // 设置上课时间索引（解析为年月日时分秒）
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
        // 动态调整日期选项
        this.updateDayOptions(year, month)
      } else {
        this.courseFormYearIndex = 0
        this.courseFormMonthIndex = 0
        this.courseFormDayIndex = 0
        this.courseFormHourIndex = 0
        this.courseFormMinuteIndex = 0
        this.courseFormSecondIndex = 0
      }
      // 设置上课地址索引
      this.courseFormLocationIndex = this.locationOptions.indexOf(item.location)
      // 设置上课节数索引
      this.courseFormStageIndex = this.stageOptions.indexOf(item.stage.toString())
      this.showModal = true
    },

    // 删除课程
    async deleteCourse(id) {
      uni.showModal({
        title: '提示',
        content: '确定删除此课程吗？',
        success: async (res) => {
          if (res.confirm) {
            try {
              const course = this.courseList.find(item => item.id === id)
              if (!course) {
                uni.showToast({ title: '课程不存在', icon: 'none' })
                return
              }
              const deleteRes = await uni.request({
                url: `http://127.0.0.1:8000/Course_manage/Delete_courses/${id}/`,
                method: 'DELETE',
                data: {
                  course_name: course.course_name, // 传递课程名称
                  class_name: course.class_name    // 传递课堂名称
                }
              })
              if (deleteRes.data.code === 200) {
                uni.showToast({ title: '删除成功', icon: 'success' })
                this.fetchAllCourses() // 刷新列表，显示所有课程
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

    // 处理表单上课时间选择变化（6列）
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

    // 处理表单上课时间列变化（动态调整日期）
    handleFormDtimeColumnChange(e) {
      const column = e.detail.column
      const value = e.detail.value
      if (column === 0) { // 年份变化
        this.courseFormYearIndex = value
        this.updateDayOptions(this.yearOptions[value], this.monthOptions[this.courseFormMonthIndex])
      } else if (column === 1) { // 月份变化
        this.courseFormMonthIndex = value
        this.updateDayOptions(this.yearOptions[this.courseFormYearIndex], this.monthOptions[value])
      }
    },

    // 处理表单上课地址选择变化
    handleFormLocationChange(e) {
      this.courseFormLocationIndex = e.detail.value
      this.courseForm.location = this.locationOptions[e.detail.value]
    },

    // 处理表单上课节数选择变化
    handleFormStageChange(e) {
      this.courseFormStageIndex = e.detail.value
      this.courseForm.stage = this.stageOptions[e.detail.value]
    },

    // 提交课程（添加/编辑）
    async submitCourse() {
      try {
        const method = this.courseForm.id ? 'PUT' : 'POST'
        const url = this.courseForm.id
          ? `http://127.0.0.1:8000/Course_manage/courses/${this.courseForm.id}/`
          : 'http://127.0.0.1:8000/Course_manage/Add_courses/'

        // 验证时间格式（使用 picker 选择，无需额外验证，但仍可保留）
        if (this.courseForm.dtime && !/^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$/.test(this.courseForm.dtime)) {
          uni.showToast({ title: '时间格式错误，请输入 YYYY-MM-DD HH:MM:SS', icon: 'none' })
          return
        }

        // 验证节数为正整数（通过 picker 选择已限制范围）
        if (this.courseForm.stage && (!/^\d+$/.test(this.courseForm.stage) || parseInt(this.courseForm.stage) <= 0)) {
          uni.showToast({ title: '节数必须为正整数', icon: 'none' })
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
            location: this.courseForm.location,
            stage: this.courseForm.stage
          }
        })
        if (res.data.code === 200) {
          uni.showToast({ title: this.courseForm.id ? '修改成功' : '添加成功', icon: 'success' })
          this.closeModal()
          this.fetchAllCourses() // 提交后刷新列表，显示所有课程
        } else if (res.data.code === 404) {
          uni.showToast({ title: this.courseForm.id ? '修改失败' : '已经有该课程', icon: 'none' })
        } else{
		  uni.showToast({ title: this.courseForm.id ? '修改失败' : '添加失败', icon: 'none' })
		}
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 关闭弹窗
    closeModal() {
      this.showModal = false
      this.courseForm = {
        id: null,
        className: '',
        courseName: '',
        teacherName: '',
        dtime: '',
        location: '',
        stage: ''
      }
      this.courseFormClassNameIndex = 0 // 重置课堂名称索引
      this.courseFormCourseNameIndex = 0 // 重置课程名称索引
      this.courseFormYearIndex = 0 // 重置年份索引
      this.courseFormMonthIndex = 0 // 重置月份索引
      this.courseFormDayIndex = 0 // 重置日期索引
      this.courseFormHourIndex = 0 // 重置小时索引
      this.courseFormMinuteIndex = 0 // 重置分钟索引
      this.courseFormSecondIndex = 0 // 重置秒钟索引
      this.courseFormLocationIndex = 0 // 重置上课地址索引
      this.courseFormStageIndex = 0 // 重置上课节数索引
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
  width: 250rpx; /* 保持输入框宽度（教师姓名） */
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
  width: 300rpx; /* 保持输入框宽度（教师姓名） */
}

.picker-text {
   position: relative; /* 设置为相对定位 */
   width: 210rpx;
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