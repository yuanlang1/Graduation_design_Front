<template>
  <view class="logind">
    <view class="profile-list">
      <view class="profile-item">
        <text class="profile-label">头像</text>
        <view class="avatar">
          <image class="avatar-img" :src="user.avatar" @tap="uploadAvatar"></image>
        </view>
      </view>

      <view class="profile-list">
        <view class="profile-item">
          <text class="profile-label">昵称</text>
          <view class="profile-action">
            <text>{{user.name}}</text>
          </view>
        </view>
      </view>

      <!-- 功能列表 -->
      <view class="profile-list">
        <view class="profile-item">
          <text class="profile-label">草莓图片</text>
          <view class="profile-action">
            <text>[点击]</text>
            <text class="emoji">😸</text>
          </view>
        </view>

        <view class="profile-item">
          <text class="profile-label">二维码名片</text>
          <image  class="qr-icon" src="../../static/label.png" mode=""></image>
        </view>

        <view class="profile-item">
          <text class="profile-label">关于我们</text>
        </view>
        
        <view class="profile-item">
          <text class="profile-label">设置</text>
        </view>
        
        <view class="profile-item">
          <text class="profile-label">系统版本</text>
          <text class="profile-value">7.0.2</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      user: {
        avatar: '../../static/001.png',
        name: "草莓",
        tel: ""
      }
    }
  },
  methods: {
    uploadAvatar() {
      console.log("修改头像----------")
      uni.chooseImage({
        count: 1,
        success: (chooseImageRes) => {
          const tempFilePaths = chooseImageRes.tempFilePaths;
          uni.uploadFile({
            url: 'http://127.0.0.1:8000/user/img/',
            filePath: tempFilePaths[0],
            name: 'avatar',
            success: (uploadFileRes) => {
              let data = JSON.parse(uploadFileRes.data)
              console.log(data, typeof(data))
              this.user.avatar = data.img
              uni.setStorageSync("user", this.user)
            }
          });
        }
      });
    },
    onshow() {
      let user = uni.getStorageSync("user")
      if (user && user.tel) {
        this.user = user
      }
    }
  }
}
</script>


<style>
.example {
  padding: 15px;
  background-color: #fff;
}

.button {
  display: flex;
  align-items: center;
  height: 35px;
  margin-left: 10px;
}

.profile-container {
  background-color: #f5f5f5;
  padding: 20rpx;
}

.profile-header {
  display: flex;
  align-items: center;
  padding: 20rpx;
  background-color: #fff;
}

.avatar {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  overflow: hidden;
  margin-right: 20rpx;
}

.avatar-img {
  width: 100%;
  height: 100%;
}

.profile-name {
  font-size: 32rpx;
  color: #333;
}

.profile-list {
  margin-top: 10rpx;
  background-color: #fff;
}

.profile-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx;
  border-bottom: 1px solid #eaeaea;
}

.profile-label {
  font-size: 28rpx;
  color: #333;
}

.profile-action,.profile-value {
  font-size: 28rpx;
  color: #888;
}

.emoji {
  margin-left: 10rpx;
}

.qr-icon,.arrow-icon {
  width: 40rpx;
  height: 40rpx;
}

.profile-item:last-child {
  border-bottom: none;
}
</style>