<template>
  <view class="container">
    <view class="title">用户注册</view>
    <view class="form">

      <view class="input-box">
        <text class="label">手机号：</text>
        <input class="input" type="number" v-model="phone" placeholder="请输入手机号" />
      </view>

      <view class="input-box">
        <text class="label">验证码：</text>
        <input class="input" type="number" v-model="code" placeholder="请输入验证码" />
        <button class="send-code" @click="sendCode" :disabled="countdown > 0">
          {{ countdown > 0 ? `${countdown}s` : '获取验证码' }}
        </button>
      </view>

      <view class="input-box">
        <text class="label">账号：</text>
        <input class="input" type="text" v-model="username" placeholder="请输入账号" />
      </view>

      <view class="input-box">
        <text class="label">密码：</text>
        <input class="input" type="password" v-model="password" placeholder="请输入密码(8~20位)" />
      </view>

      <view class="input-box">
        <text class="label">性别：</text>
        <radio-group @change="selectGender">
          <label class="radio-item">
            <radio value="男" :checked="gender === '男'" /> 男
          </label>
          <label class="radio-item">
            <radio value="女" :checked="gender === '女'" /> 女
          </label>
        </radio-group>
      </view>

      <button class="register-btn" @click="register">注册</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      phone: '',
      code: '',
      username: '',
      password: '',
      gender: '',
      countdown: 0,
      timer: null
    };
  },
  methods: {
    selectGender(e) {
      this.gender = e.detail.value;
    },
    sendCode() {
	  console.log('手机号：', this.phone);
      if (!this.phone) {
        uni.showToast({
          title: '请输入手机号',
          icon: 'none'
        });
        return;
      }
      uni.request({
        url: 'http://127.0.0.1:8000/user/send-code/',
        method: 'POST',
        data: { phone: this.phone },
        success: (res) => {
          if (res.data.code === 200) {
            uni.showToast({ title: '验证码已发送', icon: 'success' });
            this.countdown = 60;
            this.timer = setInterval(() => {
              this.countdown--;
              if (this.countdown <= 0) {
                clearInterval(this.timer);
              }
            }, 1000);
          } else {
            uni.showToast({ title: res.data.msg, icon: 'none' });
          }
        },
        fail: () => {
          uni.showToast({ title: '请求失败，请重试', icon: 'none' });
        }
      });
    },
    register() {
      if (!this.phone || !this.code || !this.username || !this.password || !this.gender) {
        uni.showToast({
          title: '请填写完整信息',
          icon: 'none'
        });
        return;
      }
	  if(this.password.length < 8 || this.password.length > 20){
		  uni.showToast({
		    title: '请确认密码的格式',
		    icon: 'none'
		  });
		  return;
	  }
      uni.request({
        url: 'http://127.0.0.1:8000/user/register/',
        method: 'POST',
        data: {
          phone: this.phone,
          code: this.code,
          username: this.username,
          password: this.password,
          gender: this.gender
        },
        success: (res) => {
          if (res.data.code === 200) {
            uni.showToast({ title: '注册成功！', icon: 'success' });
            setTimeout(() => {
              uni.navigateTo({ url: '/pages/index/index' });
            }, 1500);
          } else {
            uni.showToast({ title: res.data.msg, icon: 'none' });
          }
        },
        fail: () => {
          uni.showToast({ title: '请求失败，请重试', icon: 'none' });
        }
      });
    }
  }
};
</script>

<style scoped>
page {
  background: linear-gradient(270deg, #E1ECFE 0%, #DDFCFF 100%);
}
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 60vh;
}
.title {
  text-align: center;
  font-size: 80rpx;
  font-family: serif;
  letter-spacing: 67rpx;
  color: #26344D;
  margin-top: 20rpx;
  height: 10vh;
}
.form {
  width: 20%;
  background: white;
  padding: 40rpx;
  border-radius: 20rpx;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.1);
}
.input-box {
  display: flex;
  align-items: center;
  margin-bottom: 30rpx;
}
.label {
  width: 140rpx;
  font-size: 30rpx;
   text-align: right;
}
.radio-item{
	margin-right: 80rpx;
}
.input {
  flex: 1;
  height: 70rpx;
  border: 1px solid #ccc;
  padding: 0 20rpx;
  border-radius: 10rpx;
  font-size: 28rpx;
}
.send-code {
  margin-left: 20rpx;
  font-size: 28rpx;
  color: #007aff;
  border: none;
  background: none;
}
.register-btn {
  width: 100%;
  height: 80rpx;
  background-color: #007aff;
  color: white;
  font-size: 32rpx;
  text-align: center;
  line-height: 80rpx;
  border-radius: 10rpx;
  margin-top: 20rpx;
}
</style>
