<template>
  <view>
    <view class="content">
      <view class="sub-title">基于动作识别的课堂评估系统</view>
    </view>
    
    <view class="unlogin">
      <uni-section title="用户登录" type="line">
        <view class="example">
          <uni-forms ref="valiform" :modelValue="logindata">
            <uni-forms-item label="账号:" required name="tel">
              <uni-easyinput type="number" v-model="logindata.tel" placeholder="请输入账号" prefix-icon="person"/>
            </uni-forms-item>
            
            <uni-forms-item label="密码:" required name="passwd">
              <uni-easyinput type="password" v-model="logindata.passwd" placeholder="请输入密码" prefix-icon="locked"/>
            </uni-forms-item>
          </uni-forms>
          
          <view>
            <uni-forms-item>
              <checkbox-group>
                <label class="checkbox-label">
                  <checkbox/> 我已阅读并同意《用户协议》以及《隐私政策》
                </label>
              </checkbox-group>
            </uni-forms-item>
          </view>
          <viem style="display: flex; justify-content: center; gap: 60px;">
			  <text type="default"  @click="userforget">忘记密码</text>
			  <text type="default"  @click="userRegister">用户注册</text>
		  </viem>
          <button type="default" class="login-btn" 	@tap="userLogin">登录 </button>
          <!-- <button type="default" class="register-btn" @tap="userRegister">注册</button> -->
            <!-- <view class="register">立即注册</view> -->
        </view>
      </uni-section>
    </view>
    
  </view>
</template>

<script>
export default {
  data() {
    return {
      user: {
        avatar: "https://oimagec7.ydstatic.com/image?id=-7601312740411526213&product=xue",
        name: "",
        tel: "",
      },
      // 校验表单数据（登录和注册共用）
      logindata: {
        tel: "",
        passwd: ""
      },
      islogin: false,
    }
  },
  methods: {
    userLogin() {
      let data = this.logindata;
      console.log("登录数据:", data);
      uni.request({
        url: 'http://127.0.0.1:8000/user/information/', 
        data: data,
        method: "POST",
        success: (res) => {
          console.log("登录返回:", res.data.data);
          if (res.data.code == 200) {
            this.islogin = true;
            this.user.name = res.data.data.username;
            this.user.tel = res.data.data.phone;
            uni.setStorageSync('user', this.user);
            uni.navigateTo({
              url: '/pages/home/home'
            });
          } else {
            uni.showToast({
              title: '登录失败',
              icon: 'none'
            });
          }
        },
        fail: () => {
          uni.showToast({
            title: '请求失败',
            icon: 'none'
          });
        }
      });
    },
    userRegister() {
		uni.navigateTo({
            url: '/pages/register/register' // 确保路径正确
        });
    },
	userforget(){
		uni.navigateTo({
		    url: '/pages/userforget/userforget' // 确保路径正确
		});
	},
    onShow() {
      let user = uni.getStorageSync('user');
      if (user && user.tel) {
        this.islogin = true;
        this.user = user;
      }
    }
  },
}
</script>

<style>
page {
  background: linear-gradient(270deg, #E1ECFE 0%, #DDFCFF 100%);
}
.content {
  margin: 180rpx 0 -400rpx 0;
  width: 100%;
  text-align: center;
}
.sub-title {
  text-align: center;
  font-size: 80rpx;
  font-family: serif;
  letter-spacing: 17rpx;
  color: #26344D;
  margin-top: 10rpx;
}
.unlogin {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
.example {
  width: 90%;
  max-width: 400px;
  background-color: white;
  padding: 20px;
  border-radius: 10px;
}
.input-field {
  border: none;
  border-bottom: 1px solid #ccc;
  border-radius: 0;
}
.login-btn, .register-btn {
  width: 90%;
  margin-top: 20px;
  background-color: #5990F5;
  color: white;
  padding: 6px;
  border-radius: 5px;
  border: none;
}
.checkbox-label {
  display: flex;
  align-items: center;
  margin-top: 10px;
  font-size: 14px;
  color: #666;
}
.checkbox-label checkbox {
  margin-right: 5px;
}
.register-section {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 15px;
}
.register-section text {
  color: #666;
  font-size: 14px;
  margin-right: 5px;
}
.register-btn {
  background-color: #f0f0f0;
}
</style>
