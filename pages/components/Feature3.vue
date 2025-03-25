<template>
  <view class="container">
    <!-- 顶部查询区域 -->
    <view class="search-bar">
      <view class="search-item">
        <text>教师姓名：</text>
        <input
          type="text"
          placeholder="请输入教师姓名"
          v-model="searchParams.teacherName"
          @confirm="fetchTeachers"
        />
      </view>
      <view class="search-item">
        <text>教师性别：</text>
        <picker
          mode="selector"
          :value="searchParamsGenderIndex"
          :range="['男', '女']"
          @change="handleSearchGenderChange"
        >
          <view class="picker-text">
		  {{ searchParams.teacherGender || '请选择性别' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>教师职务：</text>
        <picker
          mode="selector"
          :value="searchParamsPositionIndex"
          :range="['任课教师', '班主任', '教导主任', '年级主任', '德育主任', '教务主任', '副校长', '校长', '无']"
          @change="handleSearchPositionChange"
        >
          <view class="picker-text">
		  {{ searchParams.teacherPosition || '请选择职务' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>教师职称：</text>
        <picker
          mode="selector"
          :value="searchParamsTitleIndex"
          :range="['三级教师', '二级教师', '一级教师', '高级教师', '正高级教师']"
          @change="handleSearchTitleChange"
        >
          <view class="picker-text">
		  {{ searchParams.teacherTitle || '请选择职称' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>教育程度：</text>
        <picker
          mode="selector"
          :value="searchParamsEducationIndex"
          :range="['小学', '高中', '本科', '硕士', '博士', '博士后']"
          @change="handleSearchEducationChange"
        >
          <view class="picker-text2">
		  {{ searchParams.teacherEducation || '请选择教育程度' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>课程名称：</text>
        <picker
          mode="selector"
          :value="searchParamsCourseNameIndex"
          :range="['数学课', '语文课', '英语课', '物理课', '生物课', '化学课', '音乐课', '体育课', '历史课', '信息技术课']"
          @change="handleSearchCourseNameChange"
        >
          <view class="picker-text2">
		  {{ searchParams.courseName || '请选择课程名称' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <view class="search-item">
        <text>教师年龄：</text>
        <picker
          mode="selector"
          :value="searchParamsAgeIndex"
          :range="ageOptions"
          @change="handleSearchAgeChange"
        >
          <view class="picker-text">
		  {{ searchParams.teacherAge || '请选择年龄' }}
		  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
		  </view>
        </picker>
      </view>
      <button type="primary" @click="fetchTeachers" style="margin-left: 10rpx; margin-right: 20rpx;">查询</button>
      <button type="primary" @click="addTeacher" style="margin-left: 30rpx;">添加教师</button>
    </view>

    <!-- 教师信息列表 -->
    <view class="table-container">
      <view class="table-header">
        <text class="table-th" style="flex: 1;">教师姓名</text>
        <text class="table-th" style="flex: 1;">性别</text>
        <text class="table-th" style="flex: 1;">职务</text>
        <text class="table-th" style="flex: 1;">职称</text>
        <text class="table-th" style="flex: 1;">教育程度</text>
        <text class="table-th" style="flex: 1;">课程名称</text>
        <text class="table-th" style="flex: 1;">年龄</text>
        <text class="table-th" style="flex: 1;">操作</text>
      </view>
      <view class="table-body" v-for="(item, index) in teacherList" :key="index">
        <text class="table-td" style="flex: 1;">{{ item.name }}</text>
        <text class="table-td" style="flex: 1;">{{ item.gender }}</text>
        <text class="table-td" style="flex: 1;">{{ item.position }}</text>
        <text class="table-td" style="flex: 1;">{{ item.title }}</text>
        <text class="table-td" style="flex: 1;">{{ item.education }}</text>
        <text class="table-td" style="flex: 1;">{{ item.courseName }}</text>
        <text class="table-td" style="flex: 1;">{{ item.age }}</text>
        <view class="table-td" style="flex: 1;">
          <button type="success" style="background-color: #45a049; color: aliceblue;" size="mini" @click="editTeacher(item)">修改</button>
          <button type="warn" size="mini" @click="deleteTeacher(item.id)">删除</button>
        </view>
      </view>
      <view v-if="teacherList.length === 0" class="no-data">暂无数据</view>
    </view>

    <!-- 添加/修改教师弹窗 -->
    <view class="modal" v-if="showModal">
      <view class="modal-content">
        <view class="modal-header">
          <text>{{ modalTitle }}</text>
          <text class="close-btn" @click="closeModal">×</text>
        </view>
        <view class="modal-body">
          <view class="form-item">
            <text>教师姓名：</text>
            <input
              type="text"
              v-model="teacherForm.name"
              placeholder="请输入教师姓名"
            />
          </view>
          <view class="form-item">
            <text>教师性别：</text>
            <picker
              mode="selector"
              :value="teacherFormGenderIndex"
              :range="['男', '女']"
              @change="handleFormGenderChange"
            >
              <view class="picker-text" style="width: 250rpx;">
			  {{ teacherForm.gender || '请选择性别' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>教师职务：</text>
            <picker
              mode="selector"
              :value="teacherFormPositionIndex"
              :range="['任课教师', '班主任', '教导主任', '年级主任', '德育主任', '教务主任', '副校长', '校长', '无']"
              @change="handleFormPositionChange"
            >
              <view class="picker-text"  style="width: 250rpx;">
			  {{ teacherForm.position || '请选择职务' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>教师职称：</text>
            <picker
              mode="selector"
              :value="teacherFormTitleIndex"
              :range="['三级教师', '二级教师', '一级教师', '高级教师', '正高级教师']"
              @change="handleFormTitleChange"
            >
              <view class="picker-text" style="width: 250rpx;">
			  {{ teacherForm.title || '请选择职称' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>教育程度：</text>
            <picker
              mode="selector"
              :value="teacherFormEducationIndex"
              :range="['小学', '高中', '本科', '硕士', '博士', '博士后']"
              @change="handleFormEducationChange"
            >
              <view class="picker-text2" style="width: 250rpx;">
			  {{ teacherForm.education || '请选择教育程度' }}
			  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>课程名称：</text>
            <picker
              mode="selector"
              :value="teacherFormCourseNameIndex"
              :range="['数学课', '语文课', '英语课', '物理课', '生物课', '化学课', '音乐课', '体育课', '历史课', '信息技术课']"
              @change="handleFormCourseNameChange"
            >
              <view class="picker-text2"  style="width: 250rpx;">
			  {{ teacherForm.courseName || '请选择课程名称' }}
			  <image class="picker-arrow" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
          <view class="form-item">
            <text>教师年龄：</text>
            <picker
              mode="selector"
              :value="teacherFormAgeIndex"
              :range="ageOptions"
              @change="handleFormAgeChange"
            >
              <view class="picker-text" style="width: 250rpx;">
			  {{ teacherForm.age || '请选择年龄' }}
			  <image class="picker-arrow2" src="/static/6723913f80304fc9a7bafc70d4dd9bd8.png" mode="widthFix" />
			  </view>
            </picker>
          </view>
        </view>
        <view class="modal-footer">
          <button type="primary" @click="submitTeacher">提交</button>
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
        teacherName: '',
        teacherGender: '',
        teacherPosition: '',
        teacherTitle: '',
        teacherEducation: '',
        courseName: '',
        teacherAge: ''
      },
      searchParamsGenderIndex: 0, // 搜索栏性别的索引
      // 移除 searchParamsTeacherNameIndex，因为教师姓名改为 input
      searchParamsPositionIndex: 0, // 搜索栏职务的索引
      searchParamsTitleIndex: 0, // 搜索栏职称的索引
      searchParamsEducationIndex: 0, // 搜索栏教育程度的索引
      searchParamsCourseNameIndex: 0, // 搜索栏课程名称的索引
      searchParamsAgeIndex: 0, // 搜索栏年龄的索引
      teacherNameOptions: [], // 教师姓名选项（动态从后端获取或初始化为常见值，供参考）
      ageOptions: Array.from({ length: 80 }, (_, i) => (i + 20).toString()), // 年龄选项（20-99岁）
      teacherList: [],
      showModal: false,
      modalTitle: '添加教师',
      teacherForm: {
        id: null,
        name: '',
        gender: '',
        position: '',
        title: '',
        education: '',
        courseName: '',
        age: ''
      },
      teacherFormGenderIndex: 0, // 表单性别的索引
      teacherFormPositionIndex: 0, // 表单职务的索引
      teacherFormTitleIndex: 0, // 表单职称的索引
      teacherFormEducationIndex: 0, // 表单教育程度的索引
      teacherFormCourseNameIndex: 0, // 表单课程名称的索引
      teacherFormAgeIndex: 0 // 表单年龄的索引
    }
  },
  onLoad() {
    // 进入页面时直接加载所有教师，并初始化教师姓名选项
    this.fetchAllTeachers()
    this.initializeTeacherNameOptions()
  },
  methods: {
    // 初始化教师姓名选项（假设从后端获取或使用默认值）
    initializeTeacherNameOptions() {
      // 模拟从后端获取教师姓名列表（实际应通过 API 获取）
      this.teacherNameOptions = ['张三', '李四', '王五', '赵六', ''] // 示例值，可扩展
      // 或者通过 fetchAllTeachers 获取后设置
      this.teacherNameOptions = this.teacherList.map(item => item.name).concat([''])
    },

    // 加载所有教师（无条件）
    async fetchAllTeachers() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/Teacher_manage/teachers',
          method: 'GET',
          data: {} // 显式传递空对象，确保无条件查询
        })
        if (res.data.code === 200) {
          this.teacherList = res.data.data
          // 更新教师姓名选项
          this.teacherNameOptions = this.teacherList.map(item => item.name).concat([''])
        } else {
          uni.showToast({ title: '加载教师失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 查询教师（带条件）
    async fetchTeachers() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/Teacher_manage/teachers',
          method: 'GET',
          data: {
            name: this.searchParams.teacherName, // 保持为输入字符串
            gender: this.searchParams.teacherGender,
            position: this.searchParams.teacherPosition,
            title: this.searchParams.teacherTitle,
            education: this.searchParams.teacherEducation,
            courseName: this.searchParams.courseName,
            age: this.searchParams.teacherAge
          } // 使用搜索条件
        })
        if (res.data.code === 200) {
          this.teacherList = res.data.data
        } else {
          uni.showToast({ title: '查询教师失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 移除 handleSearchTeacherNameChange，因为教师姓名改为 input

    // 处理搜索栏性别选择变化
    handleSearchGenderChange(e) {
      this.searchParamsGenderIndex = e.detail.value
      this.searchParams.teacherGender = ['男', '女', ''][e.detail.value]
    },

    // 处理搜索栏职务选择变化
    handleSearchPositionChange(e) {
      this.searchParamsPositionIndex = e.detail.value
      this.searchParams.teacherPosition = ['任课教师', '班主任', '教导主任', '年级主任', '德育主任', '教务主任', '副校长', '校长', '无', ''][e.detail.value]
    },

    // 处理搜索栏职称选择变化
    handleSearchTitleChange(e) {
      this.searchParamsTitleIndex = e.detail.value
      this.searchParams.teacherTitle = ['三级教师', '二级教师', '一级教师', '高级教师', '正高级教师', ''][e.detail.value]
    },

    // 处理搜索栏教育程度选择变化
    handleSearchEducationChange(e) {
      this.searchParamsEducationIndex = e.detail.value
      this.searchParams.teacherEducation = ['小学', '高中', '本科', '硕士', '博士', '博士后', ''][e.detail.value]
    },

    // 处理搜索栏课程名称选择变化
    handleSearchCourseNameChange(e) {
      this.searchParamsCourseNameIndex = e.detail.value
      this.searchParams.courseName = ['数学课', '语文课', '英语课', '物理课', '生物课', '化学课', '音乐课', '体育课', '历史课', '信息技术课', ''][e.detail.value]
    },

    // 处理搜索栏年龄选择变化
    handleSearchAgeChange(e) {
      this.searchParamsAgeIndex = e.detail.value
      this.searchParams.teacherAge = this.ageOptions[e.detail.value]
    },

    // 打开添加教师弹窗
    addTeacher() {
      this.modalTitle = '添加教师'
      this.teacherForm = {
        id: null,
        name: '',
        gender: '',
        position: '',
        title: '',
        education: '',
        courseName: '',
        age: ''
      }
      this.teacherFormGenderIndex = 0 // 重置表单性别索引
      this.teacherFormPositionIndex = 0 // 重置表单职务索引
      this.teacherFormTitleIndex = 0 // 重置表单职称索引
      this.teacherFormEducationIndex = 0 // 重置表单教育程度索引
      this.teacherFormCourseNameIndex = 0 // 重置表单课程名称索引
      this.teacherFormAgeIndex = 0 // 重置表单年龄索引
      this.showModal = true
    },

    // 修改教师
    editTeacher(item) {
      this.modalTitle = '修改教师'
      this.teacherForm = {
        id: item.id,
        name: item.name,
        gender: item.gender,
        position: item.position,
        title: item.title,
        education: item.education,
        courseName: item.courseName,
        age: item.age || ''
      }
      // 设置性别索引
      this.teacherFormGenderIndex = item.gender === '男' ? 0 : 1
      // 设置职务索引
      this.teacherFormPositionIndex = ['任课教师', '班主任', '教导主任', '年级主任', '德育主任', '教务主任', '副校长', '校长', '无'].indexOf(item.position)
      // 设置职称索引
      this.teacherFormTitleIndex = ['三级教师', '二级教师', '一级教师', '高级教师', '正高级教师'].indexOf(item.title)
      // 设置教育程度索引
      this.teacherFormEducationIndex = ['小学', '高中', '本科', '硕士', '博士', '博士后'].indexOf(item.education)
      // 设置课程名称索引
      this.teacherFormCourseNameIndex = ['数学课', '语文课', '英语课', '物理课', '生物课', '化学课', '音乐课', '体育课', '历史课', '信息技术课'].indexOf(item.courseName)
      // 设置年龄索引
      this.teacherFormAgeIndex = this.ageOptions.indexOf(item.age.toString())
      this.showModal = true
    },

    // 删除教师
    async deleteTeacher(id) {
      uni.showModal({
        title: '提示',
        content: '确定删除此教师吗？',
        success: async (res) => {
          if (res.confirm) {
            try {
              const deleteRes = await uni.request({
                url: `http://127.0.0.1:8000/Teacher_manage/Delete_teachers/${id}/`,
                method: 'DELETE'
              })
              if (deleteRes.data.code === 200) {
                uni.showToast({ title: '删除成功', icon: 'success' })
                this.fetchAllTeachers() // 刷新列表，显示所有教师
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

    // 处理表单性别选择变化
    handleFormGenderChange(e) {
      this.teacherFormGenderIndex = e.detail.value
      this.teacherForm.gender = ['男', '女'][e.detail.value]
    },

    // 处理表单职务选择变化
    handleFormPositionChange(e) {
      this.teacherFormPositionIndex = e.detail.value
      this.teacherForm.position = ['任课教师', '班主任', '教导主任', '年级主任', '德育主任', '教务主任', '副校长', '校长', '无'][e.detail.value]
    },

    // 处理表单职称选择变化
    handleFormTitleChange(e) {
      this.teacherFormTitleIndex = e.detail.value
      this.teacherForm.title = ['三级教师', '二级教师', '一级教师', '高级教师', '正高级教师'][e.detail.value]
    },

    // 处理表单教育程度选择变化
    handleFormEducationChange(e) {
      this.teacherFormEducationIndex = e.detail.value
      this.teacherForm.education = ['小学', '高中', '本科', '硕士', '博士', '博士后'][e.detail.value]
    },

    // 处理表单课程名称选择变化
    handleFormCourseNameChange(e) {
      this.teacherFormCourseNameIndex = e.detail.value
      this.teacherForm.courseName = ['数学课', '语文课', '英语课', '物理课', '生物课', '化学课', '音乐课', '体育课', '历史课', '信息技术课'][e.detail.value]
    },

    // 处理表单年龄选择变化
    handleFormAgeChange(e) {
      this.teacherFormAgeIndex = e.detail.value
      this.teacherForm.age = this.ageOptions[e.detail.value]
    },

    // 提交教师（添加或修改）
    async submitTeacher() {
      try {
        const method = this.teacherForm.id ? 'PUT' : 'POST'
        const url = this.teacherForm.id
          ? `http://127.0.0.1:8000/Teacher_manage/teachers/${this.teacherForm.id}/`
          : 'http://127.0.0.1:8000/Teacher_manage/Add_teachers/'

        // 验证年龄为正整数（虽然已通过 picker 选择，但仍可验证）
        if (this.teacherForm.age && (!/^\d+$/.test(this.teacherForm.age) || parseInt(this.teacherForm.age) <= 0)) {
          uni.showToast({ title: '年龄必须为正整数', icon: 'none' })
          return
        }

        const res = await uni.request({
          url: url,
          method: method,
          data: {
            name: this.teacherForm.name,
            gender: this.teacherForm.gender,
            position: this.teacherForm.position,
            title: this.teacherForm.title,
            education: this.teacherForm.education,
            courseName: this.teacherForm.courseName,
            age: this.teacherForm.age
          }
        })
        if (res.data.code === 200) {
          uni.showToast({ title: this.teacherForm.id ? '修改成功' : '添加成功', icon: 'success' })
          this.closeModal()
          this.fetchAllTeachers() // 提交后刷新列表，显示所有教师
        } else {
          uni.showToast({ title: this.teacherForm.id ? '修改失败' : '添加失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
      }
    },

    // 关闭弹窗
    closeModal() {
      this.showModal = false
      this.teacherForm = {
        id: null,
        name: '',
        gender: '',
        position: '',
        title: '',
        education: '',
        courseName: '',
        age: ''
      }
      this.teacherFormGenderIndex = 0 // 重置表单性别索引
      this.teacherFormPositionIndex = 0 // 重置表单职务索引
      this.teacherFormTitleIndex = 0 // 重置表单职称索引
      this.teacherFormEducationIndex = 0 // 重置表单教育程度索引
      this.teacherFormCourseNameIndex = 0 // 重置表单课程名称索引
      this.teacherFormAgeIndex = 0 // 重置表单年龄索引
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
  width: 250rpx; /* 调整输入框宽度（教师姓名） */
  height: 40rpx;
  padding: 0 10rpx;
  border: 1rpx solid #eee;
  border-radius: 4rpx;
  font-size: 28rpx;
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
.picker-text2 {
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
  max-height: 900rpx;
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

.modal-body input{
	widows: 250rpx;
}

.form-item {
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
}

.form-item text {
  width: 180rpx; /* 调整标签宽度以适应更长的字段名 */
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
  width: 250rpx; /* 保持输入框宽度（教师姓名） */
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