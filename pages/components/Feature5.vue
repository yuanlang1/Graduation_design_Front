<template>
  <view class="container">
    <!-- 主体布局：左侧空白 + 右侧用户信息 -->
    <view class="main-layout">
      <!-- 左侧空白区域（占位） -->
      <view class="left-placeholder"></view>

      <!-- 右侧用户信息展示区域 -->
      <view class="user-info">
        <view class="info-list">
          <!-- 头像 -->
          <view class="info-item">
            <text class="label">头像：</text>
            <view class="avatar-container">
              <image
                :src="userInfo.avatar || defaultAvatar"
                class="avatar"
                mode="aspectFill"
                @error="handleAvatarError"
              ></image>
            </view>
          </view>

          <!-- 用户名 -->
          <view class="info-item">
            <text class="label">用户名：</text>
            <text class="value">{{ userInfo.username || '未设置' }}</text>
          </view>

          <!-- 手机号 -->
          <view class="info-item">
            <text class="label">手机号：</text>
            <text class="value">{{ userInfo.phone || '未设置' }}</text>
          </view>

          <!-- 邮箱 -->
          <view class="info-item">
            <text class="label">邮箱：</text>
            <text class="value">{{ userInfo.email || '未设置' }}</text>
          </view>

          <!-- 职工号 -->
          <view class="info-item">
            <text class="label">职工号：</text>
            <text class="value">{{ userInfo.employeeId || '未设置' }}</text>
          </view>

          <!-- 地址 -->
          <view class="info-item">
            <text class="label">地址：</text>
            <text class="value">{{ userInfo.address || '未设置' }}</text>
          </view>

          <!-- 注册时间 -->
          <view class="info-item">
            <text class="label">注册时间：</text>
            <text class="value">{{ userInfo.registrationTime ? formatDate(userInfo.registrationTime) : '未设置' }}</text>
          </view>
        </view>
      </view>

    </view>

    <!-- 修改按钮（放在右侧用户信息下方） -->
    <view class="button-container">
      <button type="primary" @click="showEditModal = true">修改</button>
    </view>

    <!-- 用户信息修改 Modal -->
    <view class="modal" v-if="showEditModal">
      <view class="modal-content">
        <view class="modal-header">
          <text>修改用户信息</text>
          <text class="close-btn" @click="closeEditModal">×</text>
        </view>
        <view class="modal-body">
          <view class="form-container">
            <view class="form-item">
              <text class="label">用户名：</text>
              <input
                type="text"
                v-model="editUserInfo.username"
                :value="editUserInfo.username || ''"
                placeholder="请输入用户名"
                class="input"
              />
            </view>
            <view class="form-item">
              <text class="label">手机号：</text>
              <input
                type="number"
                v-model="editUserInfo.phone"
                :value="editUserInfo.phone || ''"
                placeholder="请输入手机号"
                class="input"
              />
            </view>
            <view class="form-item">
              <text class="label">邮箱：</text>
              <input
                type="email"
                v-model="editUserInfo.email"
                :value="editUserInfo.email || ''"
                placeholder="请输入邮箱"
                class="input"
              />
            </view>
            <view class="form-item">
              <text class="label">职工号：</text>
              <input
                type="text"
                v-model="editUserInfo.employeeId"
                :value="editUserInfo.employeeId || ''"
                placeholder="请输入职工号"
                class="input"
                disabled
              />
            </view>
            <view class="form-item">
              <text class="label">地址：</text>
              <input
                type="text"
                v-model="editUserInfo.address"
                :value="editUserInfo.address || ''"
                placeholder="请输入地址"
                class="input"
              />
            </view>
            <view class="form-item">
              <text class="label">头像：</text>
              <view class="avatar-upload">
                <image
                  :src="editUserInfo.avatar || defaultAvatar"
                  class="avatar"
                  mode="aspectFill"
                  @error="handleAvatarError"
                ></image>
                <button type="primary" size="mini" @click="chooseAvatar">上传头像</button>
              </view>
            </view>
          </view>
        </view>
        <view class="modal-footer">
          <button type="primary" @click="submitEdit">保存</button>
          <button type="default" @click="closeEditModal">取消</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userInfoLoaded: false,
      showEditModal: false,
      userInfo: {
        avatar: '',
        username: '',
        phone: '',
        email: '',
        employeeId: '',
        address: '',
        registrationTime: ''
      },
      editUserInfo: {
        avatar: '',
        username: '',
        phone: '',
        email: '',
        employeeId: '',
        address: ''
      },
      defaultAvatar: 'https://example.com/default-avatar.png' // 默认头像 URL
    }
  },
  onLoad() {
    this.fetchUserInfo()
  },
  methods: {
    // 处理头像加载失败
    handleAvatarError(e) {
      this.userInfo.avatar = this.defaultAvatar
      this.editUserInfo.avatar = this.defaultAvatar
    },

    // 格式化时间
    formatDate(date) {
      if (!date) return '未设置'
      return new Date(date).toLocaleString()
    },

    // 从缓存和后端获取用户信息
    async fetchUserInfo() {
      // 从缓存获取用户名和手机号
      const storedUser = uni.getStorageSync('user')
      if (storedUser && storedUser.name && storedUser.phone) {
        this.userInfo.username = storedUser.name
        this.userInfo.phone = storedUser.phone
      }

      try {
        // 从后端获取完整用户信息
        const res = await uni.request({
          url: 'http://127.0.0.1:8000/user/info/', // 替换为实际后端 API 地址
          method: 'GET',
          header: {
            'Content-Type': 'application/json',
            'Authorization': uni.getStorageSync('token') || '' // 假设需要 token 认证
          }
        })
        if (res.data.code === 200) {
          this.userInfo = {
            avatar: res.data.data.avatar || '',
            username: res.data.data.username || this.userInfo.username,
            phone: res.data.data.phone || this.userInfo.phone,
            email: res.data.data.email || '未设置',
            employeeId: res.data.data.employeeId || '未设置',
            address: res.data.data.address || '未设置',
            registrationTime: res.data.data.registrationTime || ''
          }
          // 复制到编辑数据
          this.editUserInfo = {
            avatar: this.userInfo.avatar,
            username: this.userInfo.username,
            phone: this.userInfo.phone,
            email: this.userInfo.email,
            employeeId: this.userInfo.employeeId,
            address: this.userInfo.address
          }
          this.userInfoLoaded = true
        } else {
          uni.showToast({ title: '获取用户信息失败', icon: 'none' })
          this.userInfoLoaded = false
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
        console.error('获取用户信息出错:', error)
        this.userInfoLoaded = false
      }
    },

    // 关闭用户信息修改 Modal
    closeEditModal() {
      this.showEditModal = false
      // 恢复原始数据
      this.editUserInfo = {
        avatar: this.userInfo.avatar,
        username: this.userInfo.username,
        phone: this.userInfo.phone,
        email: this.userInfo.email,
        employeeId: this.userInfo.employeeId,
        address: this.userInfo.address
      }
    },

    // 选择并上传头像
    async chooseAvatar() {
      try {
        const res = await uni.chooseImage({
          count: 1,
          sizeType: ['compressed'],
          sourceType: ['album', 'camera']
        })
        const tempFilePath = res.tempFilePaths[0]
        this.editUserInfo.avatar = tempFilePath

        const uploadRes = await uni.uploadFile({
          url: 'http://127.0.0.1:8000/user/upload-avatar/', // 替换为实际后端上传 API
          filePath: tempFilePath,
          name: 'file',
          formData: { userId: this.userInfo.id || '' }, // 假设用户有 ID
          header: {
            'Authorization': uni.getStorageSync('token') || ''
          }
        })
        const uploadData = JSON.parse(uploadRes.data)
        if (uploadData.code === 200) {
          this.editUserInfo.avatar = uploadData.data.avatar
          uni.showToast({ title: '头像上传成功', icon: 'success' })
        } else {
          uni.showToast({ title: `上传失败: ${uploadData.msg || '未知错误'}`, icon: 'none' })
          this.editUserInfo.avatar = this.defaultAvatar
        }
      } catch (error) {
        uni.showToast({ title: '上传头像出错，请重试', icon: 'none' })
        console.error('上传头像出错:', error)
        this.editUserInfo.avatar = this.defaultAvatar
      }
    },

    // 提交修改
    async submitEdit() {
      // 验证手机号格式（示例：11位数字）
      const phoneRegex = /^[0-9]{11}$/
      if (!phoneRegex.test(this.editUserInfo.phone)) {
        uni.showToast({ title: '手机号必须为11位数字', icon: 'none' })
        return
      }

      // 验证邮箱格式
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
      if (!emailRegex.test(this.editUserInfo.email)) {
        uni.showToast({ title: '邮箱格式错误', icon: 'none' })
        return
      }

      try {
        const res = await uni.request({
          url: `http://127.0.0.1:8000/user/update/${this.userInfo.id || ''}/`, // 替换为实际后端更新 API
          method: 'POST',
          data: {
            username: this.editUserInfo.username,
            phone: this.editUserInfo.phone,
            email: this.editUserInfo.email,
            address: this.editUserInfo.address,
            avatar: this.editUserInfo.avatar // 假设后端支持更新头像 URL
          },
          header: {
            'Content-Type': 'application/json',
            'Authorization': uni.getStorageSync('token') || ''
          }
        })
        if (res.data.code === 200) {
          uni.showToast({ title: '修改成功', icon: 'success' })
          this.userInfo = {
            ...this.userInfo,
            avatar: this.editUserInfo.avatar,
            username: this.editUserInfo.username,
            phone: this.editUserInfo.phone,
            email: this.editUserInfo.email,
            employeeId: this.editUserInfo.employeeId,
            address: this.editUserInfo.address
          }
          this.closeEditModal()
          // 更新缓存中的用户信息
          uni.setStorageSync('user', {
            name: this.userInfo.username,
            phone: this.userInfo.phone
          })
        } else {
          uni.showToast({ title: '修改失败', icon: 'none' })
        }
      } catch (error) {
        uni.showToast({ title: '请求失败，请检查网络', icon: 'none' })
        console.error('提交修改出错:', error)
      }
    },

    // 刷新用户信息
    refreshUserInfo() {
      this.userInfoLoaded = false
      this.fetchUserInfo()
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  height: 100%;
  padding: 20rpx;
  background-color: #f7f8fa;
}

.main-layout {
  display: flex;
  width: 100%;
  height: calc(100% - 80rpx); /* 减去按钮区域高度 */
  justify-content: flex-end; /* 右侧对齐 */
}

.left-placeholder {
  flex: 0 0 300rpx; /* 左侧空白占位区域宽度 */
  background-color: #f7f8fa; /* 与背景色一致 */
}

.user-info {
  flex: 1;
  max-width: 900rpx;
  background-color: #f0f8ff;
  border-radius: 20rpx;
  box-shadow: 0 8rpx 16rpx rgba(0, 0, 0, 0.15);
  padding: 40rpx;
  margin-bottom: 20rpx;
}

.info-list {
  width: 100%;
  display: flex;
  flex-direction: column;
}

.info-item {
  display: flex;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 2rpx solid #e0e0e0;
}

.info-item:last-child {
  border-bottom: none;
}

.label {
  font-size: 32rpx;
  color: #333;
  margin-right: 20rpx;
  width: 140rpx;
  text-align: right;
  font-weight: 500;
}

.value {
  font-size: 32rpx;
  color: #444;
  flex: 1;
  word-break: break-all;
}

.avatar-container {
  display: flex;
  align-items: center;
}

.avatar {
  width: 100rpx;
  height: 100rpx;
  border-radius: 12rpx;
  border: 2rpx solid #ddd;
}

.avatar-item {
  justify-content: flex-start;
}

.button-container {
  width: 100%;
  max-width: 900rpx;
  display: flex;
  justify-content: center;
  gap: 20rpx;
  margin-top: 20rpx;
}

.button-container button {
  width: 180rpx;
  height: 50rpx;
  line-height: 50rpx;
  font-size: 32rpx;
  background-color: #3b8cff;
  color: #fff;
  border-radius: 12rpx;
  box-shadow: 0 4rpx 8rpx rgba(59, 140, 255, 0.3);
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
  max-height: 1000rpx;
  background-color: #fff;
  border-radius: 20rpx;
  overflow: hidden;
  box-shadow: 0 8rpx 16rpx rgba(0, 0, 0, 0.15);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30rpx;
  border-bottom: 2rpx solid #e0e0e0;
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.close-btn {
  font-size: 40rpx;
  cursor: pointer;
  color: #999;
}

.modal-body {
  padding: 40rpx;
  max-height: 700rpx;
  overflow-y: auto;
}

.form-container {
  width: 100%;
}

.form-item {
  display: flex;
  align-items: center;
  margin-bottom: 25rpx;
}

.input {
  flex: 1;
  height: 45rpx;
  padding: 0 15rpx;
  border: 2rpx solid #eee;
  border-radius: 10rpx;
  font-size: 30rpx;
  background-color: #f9f9f9;
}

.avatar-upload {
  display: flex;
  align-items: center;
  gap: 20rpx;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  padding: 30rpx;
  gap: 20rpx;
  border-top: 2rpx solid #e0e0e0;
}

.modal-footer button {
  width: 140rpx;
  height: 45rpx;
  line-height: 45rpx;
  font-size: 30rpx;
  border-radius: 12rpx;
}

.no-data {
  text-align: center;
  padding: 20rpx;
  color: #999;
  font-size: 30rpx;
}
</style>