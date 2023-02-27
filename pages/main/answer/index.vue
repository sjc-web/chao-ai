<template>
	<view class="container">
		<view class="action">
			<view class="tips">
				官方接口通道
			</view>
			<view class="switch">
				<u-switch activeColor="#6c6ceb" :value="true"></u-switch>
			</view>
		</view>
		<view class="answer">
			<ad-custom v-if="day === 0" unit-id="adunit-4596fcdb2ac9b98f"></ad-custom>
			<view class="ask">{{text}}</view>
			<view class="content">
				{{message}}
			</view>
		</view>

		<view class="sheet">
			<view class="btn">
				<u-button open-type='contact' color="#6c6ceb" :plain="true" text="咨询客服(意见反馈)"></u-button>
			</view>
			<view class="btn">
				<u-button @click="handleCopy" iconColor="#ffffff" color="#6c6ceb" icon="file-text" text="复制问题答案">
				</u-button>
			</view>
		</view>
		<u-toast ref="uToast"></u-toast>
	</view>
</template>

<script>
	import {
		getSend
	} from "@/uni_modules/chao-AI/js_sdk/index.js"
	import {
		handleTime
	} from "@/uni_modules/chao-AI/js_sdk/common.js"
	export default {
		data() {
			return {
				message: '',
				text: '',
				openid: '',
				day: 0
			};
		},
		async onLoad({
			text,
			openid
		}) {
			const flag = await handleTime()
			const value = uni.getStorageSync('config');
			if (value && flag) {
				const {
					day
				} = value
				this.day = day
			}
			this.text = text
			this.openid = openid
			const that = this
			uni.showLoading({
				title: 'AI处理中',
				mask: true
			})
			const res = await getSend({
				prompt: `Q: ${text} `,
				openid,
				id: uni.getAccountInfoSync().miniProgram.appId,
			})
			uni.hideLoading()
			that.message = res
		},
		methods: {
			handleCopy() {
				const that = this
				uni.setClipboardData({
					data: that.message,
					success: function() {}
				})
			}
		},
	}
</script>

<style lang="scss">
	.answer {
		width: 90%;
		margin: 0 auto;
		margin-top: 200rpx;
		z-index: -1;

		.ask {
			font-size: 36rpx;
			font-weight: bolder;
		}

		.content {
			margin-top: 30rpx;
			color: #606266;
		}
	}

	.action {
		width: 92%;
		position: fixed;
		bottom: 0rpx;
		top: 0rpx;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		height: 80rpx;
		align-items: center;
		box-shadow: 10rpx 10rpx 10rpx #eee;
		padding: 30rpx;
		z-index: 999;
		background-color: #fff;
	}

	.sheet {
		width: 100%;
		position: fixed;
		bottom: 0rpx;
		left: 0rpx;
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		bottom: env(safe-area-inset-bottom);
		padding: 30rpx 0rpx;
		background-color: #fff;
		box-shadow: -10rpx -10rpx 10rpx #eee;

		.btn {
			width: 42%;

			.u-button {
				height: 90rpx;
			}
		}
	}
</style>
