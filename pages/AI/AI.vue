<template>
	<view class="container">
		<!-- 消息内容 -->
				<scroll-view class="chat-message" scroll-y>
					
								<!-- 正确的 key 应该放在 view 上，而不是 template 上 -->
								<view v-for="(d, ind) in userdata" :key="ind" class="message user">
									<view class="text">
										<image :src="d" mode="aspectFit"></image>
									</view>
									<image class="avatar" src="../../static/logo.png" mode=""></image>
								</view>
					
					
								<view v-for="(d, ind) in aidata" :key="ind" class="message bot">
									<image class="avatar" src="../../static/ai2.png" mode=""></image>
									<text class="text">
										<image :src="aidata[ind]" mode="aspectFit"></image>
										<text class="text">
										          熟果: {{ categoryCounts.ripe }} 未熟果: {{ categoryCounts.unripe }} 花蕾: {{ categoryCounts.flower }}  坏果: {{ categoryCounts.rotten }}
										</text>
									</text>
								</view>
								
								
				</scroll-view>

		<!-- 输入框和图标部分 -->
		<view class="input-section">
			<view class="icon-container">
				<image class="icon" src="../../static/picture.png" @tap="chooseImage"></image>
				<image class="icon" src="../../static/camera.png" @tap="camera"></image>
				<image class="icon" src="../../static/goback.png" @tap="goBack"></image>
<!-- 				<image class="icon" src="../../static/pic3.png" @tap="goBack"></image> -->
			</view>
		</view>
	</view>
</template>

<script>

	export default {
		data() {
			return {
				userdata: [],
				aidata: [],
				categoryCounts: {
				        ripe: 0,
				        unripe: 0,
				        flower: 0,
				        rotten: 0
				},  // 存储类别数量
				host: ''  // 定义 host 变量
			}
		},
		onShow() {
			// 判断当前页面是否为首页，如果不是首页才隐藏 tabBar
			if (this.$route.path !=="pages/home/home") {
				uni.hideTabBar();
			}
		},
		created() {
				this.host = 'http://127.0.0.1:8000';  // 在 created 生命周期中设置 host
			},

		methods: {
			goBack(){
				// 显示 tabBar
				uni.showTabBar();
				// 返回首页
				 uni.switchTab({
					url: "/pages/home/home"
				});
			},
			chooseImage() {
				uni.chooseImage({
					count: 1,
					success: (res) => {
						let tempFilePaths = res.tempFilePaths;
						console.log(tempFilePaths);
			
						// 定义 url 变量
						let url = this.host + '/ai/yolo/';
						console.log(url);
			
						// 上传文件
						uni.uploadFile({
							url: url,  // 使用上面定义的 url 变量
							filePath: tempFilePaths[0],
							name: 'aiimg123456',
							success: (res) => {
								try {
								        let data = JSON.parse(res.data);
								        if (data.code === 200) {
								            this.userdata.push(data.url);
								            this.aidata.push(data.exp_url);
											this.categoryCounts = data.category_counts;
								        } else {
								            console.error("Error in response:", data);
								        }
								    } catch (e) {
								        console.error("Failed to parse JSON:", e);
								    }
							}
						});
					}
				});
			},

			camera() {
				console.log('拍照......');
				uni.chooseImage({
					count: 1,
					success: function(res) {
						var tempFilePaths = res.tempFilePaths;
						console.log(tempFilePaths);
					}
				});
			}
		}
	}
</script>

<style>
	.container {
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		height: 100vh;
		padding: 40rpx;
		box-sizing: border-box;
		position: relative;
	}

	.chat-message {
		width: 100%;
		flex: 1;
		margin-bottom: 150rpx;
		/* 保证内容不会被input-section遮挡 */
		overflow-y: auto;
	}

	.message {
		display: flex;
		align-items: flex-start;
		margin: 20rpx 0;
	}

	.avatar {
		width: 80rpx;
		height: 80rpx;
		border-radius: 5rpx;
		margin-right: 20rpx;
	}
	
	.avatar2 {
		width: 750rpx;
		border-radius: 5rpx;
		margin-right: 20rpx;
	}

	.user .avatar {
		margin-left: 20rpx;
	}

	.message.user .text {
		background-color: #a4ed92;
		padding: 20rpx;
		border-radius: 10rpx;
		color: #000;
		flex: 1;
	}

	.message.bot .text {
		background-color: #e5e5e5;
		padding: 20rpx;
		border-radius: 10rpx;
		color: #000;
		flex: 1;
	}

	.text {
		display: inline-block;
	}

	.text image {
		width: 100%;
	}

	.input-section {
		width: 100%;
		max-width: 600px;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 10rpx;
		background-color: #fff;
		border-top: 1px solid #e5e5e5;
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		box-sizing: border-box;
		height: 100rpx;
		/* 根据需要调整 */
	}

	.icon-container {
		width: 100%;
		display: flex;
		justify-content: space-around;
	}

	.icon {
		width: 60rpx;
		height: 60rpx;
	}
</style>