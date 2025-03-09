<template>
  <view class="container">
    <!-- 顶部栏 -->
    <view class="header">
      <view class="header-left">
        基于动作识别的课堂评估系统
      </view>
      <view class="header-right">
        <view class="fullscreen-btn" @click="toggleFullscreen" style="padding-right: 30rpx;">全屏</view>
        <view class="user-info" @click="showUserModal">
          <image :src="getUrl(userAvatar)" class="user-avatar"></image>
          <text>{{ userName }}</text>
        </view>
      </view>
    </view>

    <!-- 主体区域：左侧菜单 + 右侧内容 -->
    <view class="body">
      <!-- 左侧菜单 -->
      <view class="sidebar">
        <view
          class="menu-item"
          v-for="(item, index) in menuList"
          :key="index"
          :class="{ active: activeMenu === index }"
          @click="selectMenu(index)"
        >
          {{ item.name }}
        </view>
      </view>

      <!-- 右侧内容区域 -->
      <view class="main-content">
        <!-- 使用动态组件展示不同功能界面 -->
        <component :is="activeComponent"></component>
      </view>
    </view>

    <!-- 用户信息弹出窗口 -->
    <view class="modal" v-if="showModal" @click="closeModal">
      <view class="modal-content" @click.stop>
        <view class="modal-header">用户信息</view>
        <view class="user-info-detail">
          <view class="avatar-container">
            <text class="label" style="padding-right: 350rpx;">头像：</text>
            <image :src="getUrl(userAvatar)" mode="aspectFill" class="avatar-img"></image>
          </view>
          <view class="info-list">
            <view class="info-item">
              <text class="label">用户名：</text>
              <text class="value">{{ userInfo.username || '未设置' }}</text>
            </view>
            <view class="info-item">
              <text class="label">手机号：</text>
              <text class="value">{{ userInfo.phone || '未设置' }}</text>
            </view>
            <view class="info-item">
              <text class="label">邮箱：</text>
              <text class="value">{{ userInfo.email || '未设置' }}</text>
            </view>
            <view class="info-item">
              <text class="label">职工号：</text>
              <text class="value">{{ userInfo.employeeId || '未设置' }}</text>
            </view>
            <view class="info-item">
              <text class="label">地址：</text>
              <text class="value">{{ userInfo.address || '未设置' }}</text>
            </view>
            <view class="info-item">
              <text class="label">性别：</text>
              <picker
                mode="selector"
                :value="genderIndex"
                :range="['男', '女']"
                @change="handleGenderChange"
                style="padding-left: 450rpx;"
              >
                <view class="picker-text">{{ userInfo.gender || '请选择性别' }}</view>
              </picker>
            </view>
            <view class="info-item">
              <text class="label">注册时间：</text>
              <text class="value">{{ formatDate(userInfo.registrationTime) }}</text>
            </view>
          </view>
          <view class="button-group">
            <button class="refresh-btn" @click="refreshUserInfo">刷新</button>
            <button class="edit-btn" @click="showEditModal = true">修改</button>
          </view>
        </view>
      </view>
    </view>

    <!-- 修改用户信息窗口 -->
    <view class="modal" v-if="showEditModal" @click="closeEditModal">
      <view class="modal-content" @click.stop>
        <view class="modal-header">修改用户信息</view>
        <view class="edit-form">
          <view class="form-item">
            <text class="label">头像：</text>
            <view class="avatar-upload">
              <image :src="getUrl(editUserInfo.avatar)" mode="aspectFill" class="avatar-img"></image>
              <button class="upload-btn" @click="chooseAvatar">上传头像</button>
            </view>
          </view>
          <view class="form-item">
            <text class="label">用户名：</text>
            <uni-easyinput
              v-model="editUserInfo.username"
              placeholder="请输入用户名"
              class="input"
            />
          </view>
          <view class="form-item">
            <text class="label">手机号：</text>
            <uni-easyinput
              type="number"
              v-model="editUserInfo.phone"
              placeholder="请输入手机号"
              class="input"
            />
          </view>
          <view class="form-item">
            <text class="label">邮箱：</text>
            <uni-easyinput
              type="email"
              v-model="editUserInfo.email"
              placeholder="请输入邮箱"
              class="input"
            />
          </view>
          <view class="form-item">
            <text class="label">职工号：</text>
            <uni-easyinput
              v-model="editUserInfo.employeeId"
              placeholder="请输入职工号"
              class="input"
            />
          </view>
          <view class="form-item">
            <text class="label">地址：</text>
            <uni-easyinput
              v-model="editUserInfo.address"
              placeholder="请输入地址"
              class="input"
            />
          </view>
          <view class="form-item">
            <text class="label">性别：</text>
            <picker
              mode="selector"
              :value="editGenderIndex"
              :range="['男', '女']"
              @change="handleEditGenderChange"
            >
              <view class="picker-text" style="width: 400;">{{ editUserInfo.gender || '请选择性别' }}</view>
            </picker>
          </view>
          <view class="form-item">
            <text class="label">注册时间：</text>
            <text class="value">{{ formatDate(editUserInfo.registrationTime) }}</text>
          </view>
        </view>
        <view class="button-group">
          <button class="save-btn" @click="submitEdit">保存</button>
          <button class="cancel-btn" @click="closeEditModal">取消</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import Overview from '@/pages/components/Overview.vue';
import Feature1 from '@/pages/components/Feature1.vue';
import Feature2 from '@/pages/components/Feature2.vue';
import Feature3 from '@/pages/components/Feature3.vue';
import Feature4 from '@/pages/components/Feature4.vue';

export default {
  data() {
    return {
      userName: 'Admin', // 默认用户名
      userAvatar: '', // 用户头像URL
      userphone: '',
      showModal: false, // 控制用户信息弹出窗口显示
      showEditModal: false, // 控制修改窗口显示
      userInfo: {
        avatar: '',
        username: '',
        phone: '',
        email: '',
        employeeId: '',
        address: '',
        gender: '', // 添加性别字段
        registrationTime: ''
      },
      editUserInfo: {
        avatar: '',
        username: '',
        phone: '',
        email: '',
        employeeId: '',
        address: '',
        gender: '',
        registrationTime: ''
      },
      menuList: [
        { name: '数据检测大屏' },
        { name: '课堂管理' },
        { name: '教师管理' },
        { name: '课程管理' }
      ],
      activeMenu: 0,
      genderIndex: 0, // 用户信息窗口性别选择索引
      editGenderIndex: 0 // 修改窗口性别选择索引
    };
  },
  onLoad() {
    const storedUser = uni.getStorageSync('user');
    console.log("数据:", storedUser.name);
    console.log("电话:", storedUser.tel);
    if (storedUser) {
      this.userName = storedUser.name;
      this.userphone = storedUser.tel;
    }
    this.fetchUserInfo();
  },
  computed: {
    activeComponent() {
      const components = [Feature1, Feature2, Feature3, Feature4];
      return components[this.activeMenu] || Overview;
    }
  },
  methods: {
    getUrl(path) {
      return `http://127.0.0.1:8000/static/rawpic/${path.replace(/^\//, '')}`;
    },
    formatDate(date) {
      if (!date) return '无';
      return new Date(date).toLocaleString();
    },
    selectMenu(index) {
      this.activeMenu = index;
    },
    showUserModal() {
      this.showModal = true; // 显示用户信息弹出窗口
      // 初始化性别选择索引
      this.genderIndex = this.userInfo.gender === '男' ? 0 : (this.userInfo.gender === '女' ? 1 : 0);
    },
    closeModal() {
      this.showModal = false; // 点击背景关闭用户信息弹出窗口
    },
    toggleFullscreen() {
      // H5平台全屏实现
      if (document.documentElement.requestFullscreen) {
        document.documentElement.requestFullscreen();
      } else if (document.documentElement.mozRequestFullScreen) {
        document.documentElement.mozRequestFullScreen();
      } else if (document.documentElement.webkitRequestFullscreen) {
        document.documentElement.webkitRequestFullscreen();
      } else if (document.documentElement.msRequestFullscreen) {
        document.documentElement.msRequestFullscreen();
      }
    },
    async fetchUserInfo() {
      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/user/get_all/',
          method: 'POST',
          data: {
            phone: this.userphone
          }
        });
        if (res.data.code === 200) {
          console.log("data:", res.data.data);
          this.userName = res.data.data.username;
          this.userInfo = {
            avatar: res.data.data.avatar,
            username: res.data.data.username,
            phone: res.data.data.phone,
            email: res.data.data.email,
            employeeId: res.data.data.employeeId,
            address: res.data.data.address,
            gender: res.data.data.gender,
            registrationTime: res.data.data.registrationTime
          };
          this.userAvatar = this.userInfo.avatar;
          this.editUserInfo = { ...this.userInfo }; 
          this.editGenderIndex = this.editUserInfo.gender === '男' ? 0 : (this.editUserInfo.gender === '女' ? 1 : 0);
          console.log("Url:", res.data.data.avatar);
        } else {
          console.log('code', res.data.code);
          uni.showToast({ title: '获取用户信息失败', icon: 'none' });
        }
      } catch (error) {
        uni.showToast({ title: '网络请求失败', icon: 'none' });
      }
    },
    handleGenderChange(e) {
      this.genderIndex = e.detail.value;
      this.userInfo.gender = ['男', '女'][this.genderIndex];
    },
    handleEditGenderChange(e) {
      this.editGenderIndex = e.detail.value;
      this.editUserInfo.gender = ['男', '女'][this.editGenderIndex];
    },
    refreshUserInfo() {
      this.fetchUserInfo(); // 刷新用户信息
    },
    showEditModal() {
      this.showEditModal = true; // 显示修改窗口
    },
    closeEditModal() {
      this.showEditModal = false; // 关闭修改窗口
      this.editUserInfo = { ...this.userInfo }; // 恢复原始数据
      this.editGenderIndex = this.userInfo.gender === '男' ? 0 : (this.userInfo.gender === '女' ? 1 : 0);
    },
    async chooseAvatar() {
      try {
        const res = await uni.chooseImage({
          count: 1,
          sizeType: ['compressed'],
          sourceType: ['album', 'camera']
        });
        const tempFilePath = res.tempFilePaths[0];
        this.editUserInfo.avatar = tempFilePath;

        const uploadRes = await uni.uploadFile({
          url: 'http://127.0.0.1:8000/Class_manage/Upload_image/',
          filePath: tempFilePath,
          name: 'file',
          formData: { phone: this.userInfo.phone },
        });
        const uploadData = JSON.parse(uploadRes.data);
        if (uploadData.code === 200) {
          this.editUserInfo.avatar = uploadData.data.url;
          uni.showToast({ title: '头像上传成功', icon: 'success' });
        } else {
          uni.showToast({ title: `上传失败: ${uploadData.msg || '未知错误'}`, icon: 'none' });
          this.editUserInfo.avatar = this.userInfo.avatar;
        }
      } catch (error) {
        uni.showToast({ title: '上传头像出错，请重试', icon: 'none' });
        console.error('上传头像错误:', error);
      }
    },
    async submitEdit() {
      // 简单验证
      if (!this.editUserInfo.username) {
        uni.showToast({ title: '用户名不能为空', icon: 'none' });
        return;
      }
      if (!this.editUserInfo.phone || !/^[0-9]{11}$/.test(this.editUserInfo.phone)) {
        uni.showToast({ title: '手机号必须为11位数字', icon: 'none' });
        return;
      }

      try {
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/user/user_updata/',
          method: 'POST',
          data: {
			avatar: this.editUserInfo.avatar,
			username: this.editUserInfo.username,
			phone: this.editUserInfo.phone,
			email: this.editUserInfo.email,
			employeeId: this.editUserInfo.employeeId,
			address: this.editUserInfo.address,
			gender: this.editUserInfo.gender
          }
        });
        if (res.data.code === 200) {
          // console.log("data:", res.data.data);
          // this.userName = res.data.data.username;
          // this.userInfo = {
          //   avatar: res.data.data.avatar,
          //   username: res.data.data.username,
          //   phone: res.data.data.phone,
          //   email: res.data.data.email,
          //   employeeId: res.data.data.employeeId,
          //   address: res.data.data.address,
          //   gender: res.data.data.gender,
          //   registrationTime: res.data.data.registrationTime
          // };
          // this.userAvatar = this.userInfo.avatar;
          // this.editUserInfo = { ...this.userInfo }; // 初始化修改数据
          // this.editGenderIndex = this.editUserInfo.gender === '男' ? 0 : (this.editUserInfo.gender === '女' ? 1 : 0);
          // console.log("Url:", res.data.data.avatar);
		  this.userphone = res.data.data.phone
		  uni.showToast({ title: '修改成功', icon: 'none' });
        } else {
          console.log('code', res.data.code);
          uni.showToast({ title: '', icon: 'none' });
        }
      } catch (error) {
        uni.showToast({ title: '网络请求失败', icon: 'none' });
      }
    }
  },
  onShow() {
    this.fetchUserInfo();
  }
};
</script>

<style scoped>
/* 整体容器 */
.container {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

/* 顶部栏 */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 80rpx;
  background-color: #3b8cff;
  color: #fff;
  padding: 0 20rpx;
}
.header-left {
  font-size: 32rpx;
  font-weight: bold;
}
.header-right {
  display: flex;
  align-items: center;
  gap: 20rpx;
}
.user-info {
  display: flex;
  align-items: center;
  cursor: pointer;
}
.user-avatar {
  width: 60rpx;
  height: 60rpx;
  border-radius: 50%;
  margin-right: 10rpx;
}
.fullscreen-btn {
  font-size: 28rpx;
  cursor: pointer;
}

/* 主体区域 */
.body {
  display: flex;
  flex: 1;
  overflow: hidden;
  justify-content: flex-end; /* 右侧内容靠最右对齐 */
}

/* 左侧菜单 */
.sidebar {
  width: 240rpx;
  background-color: #f7f8fa;
  padding: 20rpx 0;
  overflow-y: auto;
}
.menu-item {
  padding: 20rpx;
  font-size: 33rpx;
  color: #333;
  cursor: pointer;
}
.menu-item:hover {
  background-color: #e6f7ff;
}
.menu-item.active {
  background-color: #bae7ff;
  color: #1890ff;
}

/* 右侧主内容 */
.main-content {
  flex: 1;
  padding: 20rpx;
  overflow-y: auto;
  background-color: #fff;
  min-width: 0; /* 防止内容溢出 */
  max-width: calc(100vw - 240rpx); /* 限制宽度为视口宽度减去左侧菜单宽度 */
}

/* 弹出窗口样式（用户信息和修改窗口共用） */
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
  background-color: #fff;
  padding: 20rpx;
  border-radius: 10rpx;
  width: 700rpx;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.1);
}
.modal-header {
  color: #333;
  text-align: center;
  padding: 15rpx;
  font-size: 32rpx;
  font-weight: bold;
  border-bottom: 1rpx solid #eee;
  border-radius: 10rpx 10rpx 0 0;
}
.user-info-detail, .edit-form {
  padding: 20rpx;
}
.avatar-container {
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
}
.label {
  width: 160rpx;
  font-weight: bold;
  font-size: 28rpx;
  color: #333;
  text-align: justify;
}
.avatar-img {
  width: 150rpx;
  height: 100rpx;
  border-radius: 5rpx;
  border: 1rpx solid #ddd;
  object-fit: cover;
}
.info-list, .form-item {
  margin-bottom: 15rpx;
}
.info-item {
  display: flex;
  align-items: center;
}
.value {
  flex: 1;
  font-size: 28rpx;
  color: #666;
  text-align: right;
}
.picker-text {
  padding: 10rpx;
  border: 1rpx solid #ddd;
  border-radius: 5rpx;
  font-size: 28rpx;
  color: #666;
  width: 40rpx;
  text-align: left;
}
.avatar-upload {
  display: flex;
  align-items: center;
  gap: 10rpx;
}
.upload-btn {
 width: 120rpx;
 height: 40rpx;
 line-height: 40rpx;
 font-size: 28rpx;
 background-color: #1890ff;
 color: #fff;
}
.input {
  padding: 10rpx;
  border: 1rpx solid #ddd;
  border-radius: 5rpx;
  font-size: 28rpx;
  color: #666;
  width: 400rpx; /* 统一长度 */
  text-align: left;
}
.button-group {
  display: flex;
  justify-content: flex-end;
  padding: 20rpx;
  gap: 20rpx;
  border-top: 1rpx solid #eee;
}
.refresh-btn, .edit-btn {
  background-color: #1890ff;
  color: #fff;
  width: 150rpx;
  height: 60rpx;
  font-size: 28rpx;
  border-radius: 5rpx;
}
/* .save-btn {
  background-color: #1890ff;
  color: #fff;
  width: 150rpx;
  height: 60rpx;
  font-size: 28rpx;
  border-radius: 5rpx;
}
.cancel-btn {
  background-color: #fff;
  color: #1890ff;
  border: 1rpx solid #1890ff;
  width: 150rpx;
  height: 60rpx;
  font-size: 28rpx;
  border-radius: 5rpx;
} */
.button-group button{
  width: 120rpx;
  height: 40rpx;
  line-height: 40rpx;
  font-size: 28rpx;
  background-color: #1890ff;
  color: #fff;
}
</style>