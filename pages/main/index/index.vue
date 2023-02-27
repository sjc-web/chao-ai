<template>
	<view class="container">
		<view class="form">
			<view class="logo">
				<u--image src="https://wx.qlogo.cn/mmhead/Q3auHgzwzM55kOdw21icYsGmEojHmk3XMqRLWIZPkZJ6uBLvqdf6SxA/0"
					shape="circle" width="200rpx" height="200rpx">
				</u--image>
			</view>
			<u-transition :show="true" mode="slide-left">
				<view class="title">一键AI问答</view>
			</u-transition>
			<u-transition :show="true" mode="slide-right">
				<view class="desc">知你困境，解你烦恼，AI帮你写答案！</view>
			</u-transition>
			<view class="btn-group">
				<view class="btn" v-on:click="onToForm">
					<u-button shape="circle" iconColor="#ffffff" color="#6c6ceb" icon="edit-pen" text="AI问答">
					</u-button>
				</view>
				<view class="btn" v-on:click="onToDuihua">
					<u-button shape="circle" iconColor="#ffffff" color="#6c6ceb" icon="edit-pen" text="AI对话">
					</u-button>
				</view>
				<view class="btn" v-on:click="onToDuihuas">
					<u-button shape="circle" iconColor="#ffffff" color="#6c6ceb" icon="edit-pen" text="AI对话(已注册)">
					</u-button>
				</view>
				<view class="btn" v-on:click="onToWeb">
					<u-button shape="circle" iconColor="#ffffff" color="#6c6ceb" text="注册攻略">
					</u-button>
				</view>
				<view v-if="admin" class="btn" v-on:click="onToAdmin">
					<u-button shape="circle" iconColor="#ffffff" color="#6c6ceb" text="管理后台">
					</u-button>
				</view>
				<view class="btn">
					<u-button open-type='contact' color="#6c6ceb" :plain="true" text="咨询客服(意见反馈)"></u-button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		handleConfig,
		handleTime
	} from "@/uni_modules/chao-AI/js_sdk/common.js"
	export default {
		data() {
			return {
				admin: false
			};
		},
		async onLoad() {
			const flag = await handleTime()
			const value = uni.getStorageSync('config');
			const that = this
			if (value && flag) {
				const {
					admin
				} = value
				that.admin = admin
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
		onShareAppMessage: function() {
			return {
				title: '一键AI问答',
				imageUrl: '/static/wenda.png',
			};
		},
		onShareTimeline: function() {
			return {
				title: '一键AI问答',
				imageUrl: '/static/wenda.png',
			};
		},
		methods: {
			dianyingpiao() {
				uni.navigateToMiniProgram({
					appId: 'wx6c405b6b53d74d46',
					success(res) {
						// 打开成功
					}
				})

			},
			muyunianzhu() {
				uni.navigateToMiniProgram({
					appId: 'wxc0e877ab5c03d468',
					success(res) {
						// 打开成功
					}
				})

			},
			async handlefig(code) {
				const res = await handleConfig(code)
				const {
					admin
				} = res
				this.admin = admin
				uni.hideLoading()
			},
			onToForm() {
				uni.navigateTo({
					url: '/pages/main/form/index'
				})
			},
			onToDuihua() {
				uni.navigateTo({
					url: '/pages/main/duihua/duihua'
				})
			},
			onToDuihuas() {
				uni.navigateTo({
					url: '/pages/main/duihuas/duihuas'
				})
			},
			onToAdmin() {
				uni.navigateTo({
					url: '/pages/admin/index'
				})
			},
			onToWeb() {
				uni.navigateTo({
					url: '/pages/main/webview/webview?url=' + encodeURIComponent(
						'https://mp.weixin.qq.com/s?__biz=MzUzMDkzMzIxNA==&mid=2247484359&idx=1&sn=0bb59f3ca5e4e8f20a3e604a8ead5160&chksm=fa4b7776cd3cfe60fb292d4b0aa7d747c4072c130636004706ab2b6457eb5834752bf20210fd&token=1419762871&lang=zh_CN#rd'
					)
				})
			},
		}
	}
</script>

<style lang="scss">
	.form {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		margin-top: 30%;

		.title {
			font-size: 38rpx;
			font-weight: bolder;
			margin-top: 15rpx;
		}

		.desc {
			margin-top: 15rpx;
			font-size: 28rpx;
			color: #666;
		}

		.btn-group {
			width: 80%;

			.btn {
				margin: 30rpx 0rpx;

				.u-button {
					height: 100rpx;
				}
			}
		}
	}
</style>
