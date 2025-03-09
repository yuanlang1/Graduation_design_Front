<template>
  <view class="card">
    <text class="card-title">功能 1 界面</text>
    <view class="card-body">
      <text>这里是功能 1111 的具体内容</text>
    </view>
  </view>
</template>

<script>
export default {
  name: 'Feature1'
}
</script>

<style scoped>
.card {
  background-color: #fff;
  margin-bottom: 20rpx;
  padding: 20rpx;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.1);
}
.card-title {
  font-size: 32rpx;
  font-weight: bold;
  margin-bottom: 10rpx;
}
.card-body {
  font-size: 28rpx;
  color: #666;
}
</style>
