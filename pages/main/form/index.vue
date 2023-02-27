<template>
	<view class="container">
		<view class="bg"></view>
		<view class="form">
			</u-notice-bar>
			<view class="header">
				<view class="title">
					<view>人工智能回答你需要的问题</view>
					<view>免费额度{{freePerDay}}次/剩余{{ isnum }}次</view>
				</view>
			</view>
			<u-transition :show="true" mode="fade-left">
				<view class="panel">
					<u-cell isLink :border="false" title="填写您的问题" :value="isvalue" clickable @click="handleopen">
					</u-cell>
					<u-picker :show="picshow" :columns="columns" keyName="label" @cancel="handlecancel"
						@confirm="handleconfirm"></u-picker>
					<view class="textarea">
						<u--textarea v-model="text" height="140" placeholder="请输入您的问题"></u--textarea>
					</view>
					<view class="btn-group">
						<view class="btn paste">
							<u-button icon="file-text" text="粘贴问题" @click="handlezt"></u-button>
						</view>
						<view class="btn get">
							<u-button @click="onSubmitGPT" iconColor="#ffffff" color="#6c6ceb" icon="edit-pen"
								text="获取问题答案"></u-button>
						</view>
					</view>
				</view>
			</u-transition>
		</view>
		<view class="alert">
			<u-alert title="当前使用人数过多,获取答案可能超过30秒钟" type="error"></u-alert>
		</view>
		<view class="tool">
			<view class="header">
				<view class="title">功能特性</view>
			</view>
			<u-transition :show="true" mode="slide-up">
				<view class="grid">
					<view class="item">
						<view class="icon">
							<u-icon size="28" name="/static/1.png">
							</u-icon>
						</view>
						<view class="title">AI回答</view>
						<view class="desc">轻松获取</view>
					</view>
					<view class="item">
						<view class="icon">
							<u-icon size="28" name="/static/2.png">
							</u-icon>
						</view>
						<view class="title">智能答案</view>
						<view class="desc">免去千篇一律</view>
					</view>
					<view class="item">
						<view class="icon">
							<u-icon size="28" name="/static/3.png">
							</u-icon>
						</view>
						<view class="title">自动代码</view>
						<view class="desc">免去繁琐搜索</view>
					</view>
					<view class="item">
						<view class="icon">
							<u-icon size="28" name="/static/4.png">
							</u-icon>
						</view>
						<view class="title">引导写作</view>
						<view class="desc">AI发散思维</view>
					</view>
					<view class="item">
						<view class="icon">
							<u-icon size="28" name="/static/5.png">
							</u-icon>
						</view>
						<view class="title">存在限制</view>
						<view class="desc">会有偏见内容</view>
					</view>
					<view class="item">
						<view class="icon">
							<u-icon size="28" name="/static/6.png">
							</u-icon>
						</view>
						<view class="title">在线工具</view>
						<view class="desc">无需下载软件</view>
					</view>
				</view>
			</u-transition>
		</view>
		<view class="share">
			<u-button openType='share' shape="circle" color="#6c6ceb" :plain="true" icon="share">
				{{'邀请新朋友+'+invite}}
			</u-button>
		</view>
		<view class="btnss">
			<u-button shape="circle" :plain="true" color="#6c6ceb" @click="handleAd">
				{{'观看广告+'+advertisement}}
			</u-button>
		</view>
		<view class="btns">
			<u-button @click="handleCopy" shape="circle" :plain="true" color="#6c6ceb" icon="file-text" text="复制用户ID">
			</u-button>
		</view>
		<u-toast ref="uToast"></u-toast>
	</view>
</template>

<script>
	import {
		checkText,
		AddCount
	} from "@/uni_modules/chao-AI/js_sdk/index.js"
	import {
		handleConfig,
		handleTime
	} from "@/uni_modules/chao-AI/js_sdk/common.js"
	// 在页面中定义激励视频广告
	let videoAd = null
	// 在页面中定义插屏广告
	let interstitialAd = null
	export default {
		data() {
			return {
				picshow: false,
				text: '',
				isvalue: '标准线路',
				columns: [
					[{
						id: '1',
						type: '',
						label: '标准线路',
					}]
				],
				isday: 0,
				isnum: 5,
				invite: 0,
				freePerDay: 0,
				advertisement: 0,
				openid: '',
				obj: {},
				id: ''
			};
		},
		async onLoad({
			id
		}) {
			const that = this
			that.id = id
			// 在页面onLoad回调事件中创建激励视频广告实例
			if (wx.createRewardedVideoAd) {
				videoAd = wx.createRewardedVideoAd({
					adUnitId: 'adunit-593140983fbc3c32'
				})
				videoAd.onLoad(() => {})
				videoAd.onError((err) => {
					uni.showToast({
						title: '暂未开放，请邀请好友',
						icon: 'none',
						duration: 2000,
					});
				})
				videoAd.onClose((res) => {
					if (res.isEnded) {
						that.handleAdd(that.openid, that.advertisement)
						return
					} else {
						uni.showToast({
							title: '请不要中途退出!',
							icon: 'none',
							duration: 2000,
						});
					}
				})
			}
			// 在页面onLoad回调事件中创建插屏广告实例
			if (wx.createInterstitialAd) {
				interstitialAd = wx.createInterstitialAd({
					adUnitId: 'adunit-95b8a52407a3d13e'
				})
				interstitialAd.onLoad(() => {})
				interstitialAd.onError((err) => {})
				interstitialAd.onClose(() => {})
			}
			const flag = await handleTime()
			const value = uni.getStorageSync('config');
			if (value && flag) {
				const {
					day,
					freePerDay,
					invite,
					num,
					openid,
					advertisement
				} = value
				this.isday = day
				this.isnum = num
				this.invite = invite
				this.freePerDay = freePerDay
				this.advertisement = advertisement
				this.openid = openid
				this.obj = value
				return
			}
			uni.login({
				provider: "weixin",
				success(res) {
					uni.showLoading({
						title: '加载中',
						mask: true
					})
					that.handlefig(res.code)
				}
			})
		},
		onShow() {
			const that = this
			setTimeout(() => {
				// 在适合的场景显示插屏广告
				if (interstitialAd && that.isday === 0) {
					interstitialAd.show().catch((err) => {
						console.error(err)
					})
				}
			}, 1000)
		},
		onShareAppMessage: function(options) {
			const that = this;
			// 设置菜单中的转发按钮触发转发事件时的转发内容
			const shareObj = {
				title: '一键AI问答',
				imageUrl: '/static/wenda.png', //自定义图片路径，可以是本地文件路径、代码包文件路径或者网络图片路径，支持PNG及JPG，不传入 imageUrl 则使用默认截图。显示图片长宽比是 5:4
				path: '/pages/main/form/index?id=' + that.openid,
			};
			// 返回shareObj
			return shareObj;
		},
		onShareTimeline: function() {
			const that = this;
			// 设置菜单中的转发按钮触发转发事件时的转发内容
			const shareObj = {
				title: '一键AI问答',
				imageUrl: '/static/wenda.png', //自定义图片路径，可以是本地文件路径、代码包文件路径或者网络图片路径，支持PNG及JPG，不传入 imageUrl 则使用默认截图。显示图片长宽比是 5:4
				path: '/pages/main/form/index?id=' + that.openid,
			};
			// 返回shareObj
			return shareObj;
		},
		methods: {
			handleAd() {
				if (videoAd) {
					videoAd.show().catch(() => {
						// 失败重试
						videoAd.load()
							.then(() => videoAd.show())
							.catch(err => {
								console.log('激励视频 广告显示失败')
							})
					})
					return
				}
			},
			async handleAdd(id, count) {
				const that = this
				const res = await AddCount({
					openid: id,
					count: count
				})
				that.isnum = that.isnum + count
				uni.setStorageSync('config', {
					...that.obj,
					num: that.isnum
				});
				uni.showToast({
					title: '恭喜成功获取使用额度',
					icon: 'none',
					duration: 2000,
				});
			},
			async handlefig(code) {
				const res = await handleConfig(code)
				const {
					day,
					freePerDay,
					invite,
					num,
					openid,
					advertisement
				} = res
				this.isday = day
				this.isnum = num
				this.invite = invite
				this.freePerDay = freePerDay
				this.advertisement = advertisement
				this.openid = openid
				uni.hideLoading()
				this.obj = res
				if (!this.id) return
				await this.handleAdd(this.id, invite)
			},
			handleCopy() {
				const that = this
				uni.setClipboardData({
					data: that.openid,
					success: function() {}
				})
			},
			handleopen() {
				this.picshow = true
			},
			handlecancel() {
				this.picshow = false
			},
			handleconfirm(data) {
				if (data.value[0].type === 'url') {
					uni.navigateTo({
						url: '/pages/main/webview/webview?url=' + encodeURIComponent(data.value[0].url)
					})
					return
				}
				if (data.value[0].type === 'app') {
					uni.navigateToMiniProgram({
						appId: data.value[0].url,
						success(res) {
							// 打开成功
						}
					})
					return
				}
				this.picshow = false
				this.isvalue = data.value[0].label
			},
			async handlelog() {
				const that = this
				if (that.isnum < 1) {
					uni.showToast({
						title: '免费次数已用完,请观看广告或者邀请好友+次数',
						duration: 3000,
						mask: true,
						icon: 'none'
					});
					return
				}
				if (that.isday === 0) {
					that.isnum = that.isnum - 1
					uni.setStorageSync('config', {
						...that.obj,
						num: that.isnum
					});
				}
				uni.showLoading({
					title: '文本安全检测中',
					mask: true
				})
				const resa = await checkText({
					params: {
						prompt: that.text,
						openid: that.openid,
						id: uni.getAccountInfoSync().miniProgram.appId,
					},
					actions: 'msgSecCheck',
				})
				const newresult = [];
				resa.data.detail.filter(async item => {
					if (item.label === 100) {
						newresult.push(item);
						return;
					}
				});
				if (newresult.length === 0) return uni.showToast({
					title: '存在敏感词汇，请更换重新输入',
					duration: 3000,
					mask: true,
					icon: 'none'
				});
				uni.hideLoading()
				if (resa.data.errcode === 0) {
					uni.navigateTo({
						url: '/pages/main/answer/index?text=' + that.text + '&openid=' + that.openid
					})
				}
			},
			handlezt() {
				const that = this
				uni.getClipboardData({
					success: function(res) {
						that.text = res.data
					}
				})
			},
			onSubmitGPT() {
				if (this.text.length === 0) return this.$refs.uToast.show({
					type: 'default',
					message: "请输入问题"
				})
				this.handlelog()
			}
		}
	}
</script>

<style lang="scss" scoped>
	/deep/ .u-cell__body {
		padding: 0 !important;
	}

	.share {
		position: fixed;
		right: 40rpx;
		bottom: 100rpx;
		width: 45%;

		.u-button {
			box-shadow: 0rpx 10rpx 10rpx #eee !important;
		}
	}

	.btns {
		position: fixed;
		right: 40rpx;
		bottom: 220rpx;
		width: 45%;

		.u-button {
			box-shadow: 0rpx 10rpx 10rpx #eee !important;
		}
	}

	.btnss {
		position: fixed;
		right: 40rpx;
		bottom: 340rpx;
		width: 45%;

		.u-button {
			box-shadow: 0rpx 10rpx 10rpx #eee !important;
		}
	}

	.isad {

		// position: fixed;
		// right: 40rpx;
		// bottom: 180rpx;
		// width: 40%;
		.u-button {
			box-shadow: 0rpx 10rpx 10rpx #eee !important;
		}
	}

	.tool {
		width: 90%;
		margin: 15rpx auto;

		.header {
			.title {
				margin-bottom: 30rpx;
			}
		}

		.grid {
			display: flex;
			flex-direction: row;
			flex-wrap: wrap;
			justify-content: space-between;

			.item {
				width: 30%;
				display: flex;
				flex-direction: column;
				align-items: center;
				background-color: #e8eafe;
				margin: 15rpx 0rpx;
				padding: 30rpx 0rpx;
				border-radius: 10rpx;

				.title {
					color: #1B2B20;
					margin-top: 15rpx;
				}

				.desc {
					color: #67776C;
					font-size: 24rpx;
				}
			}
		}
	}

	.alert {
		width: 90%;
		margin: 30rpx auto;
	}

	.form {
		width: 90%;
		margin: 0 auto;
		z-index: 999;

		.header {
			margin-bottom: 30rpx;

			.title {
				text-align: center;
				color: #fff;
				margin: 30rpx 0rpx;
				display: flex;
				justify-content: space-between;
			}
		}

		.panel {
			padding: 30rpx;
			background-color: #fff;
			border-radius: 15rpx;
			box-shadow: 0rpx 10rpx 10rpx #eee;

			// .head {
			// 	display: flex;
			// 	flex-direction: row;
			// 	justify-content: space-between;

			// 	.tips {
			// 		color: #dd6161;
			// 	}
			// }

			.textarea {
				margin-top: 30rpx;
			}

			.btn-group {
				display: flex;
				flex-direction: row;
				justify-content: space-between;
				margin-top: 30rpx;

				.paste {
					width: 40%;
				}

				.get {
					width: 56%;
				}
			}
		}
	}

	.bg {
		position: fixed;
		top: 0rpx;
		left: 0rpx;
		width: 100%;
		background-color: #6c6ceb;
		min-height: 200rpx;
		border-radius: 0rpx 0rpx 140rpx 140rpx;
		z-index: -1;
	}
</style>
