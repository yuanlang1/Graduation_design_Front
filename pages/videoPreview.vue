<template>
  <view class="container">
    <video
      :src="videoSrc"
      controls
      autoplay
      style="width: 100%; height: 100vh;"
    ></video>
  </view>
</template>

<script>
export default {
  data() {
    return {
      videoSrc: ''
    }
  },
  onLoad(options) {
    if (options.src) {
      this.videoSrc = decodeURIComponent(options.src);
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
  background-color: #000;
}
</style>