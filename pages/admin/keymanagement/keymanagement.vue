<template>
	<view>
		<u-notice-bar bgColor='#6c6ceb' color="#fff" text="如果异常请检查是否欠费,如果强制开启,检测异常会自动关闭" mode="closable">
		</u-notice-bar>
		<u-cell-group>
			<u-cell v-for="(item, index) in KeyList" :key="item._id">
				<view slot="title" class="u-slot-title">
					<text class="u-cell-text">{{$u.timeFormat(item.create_data, 'yyyy-mm-dd')}}</text>
					<u-tag v-if="item.state" text="正常" type="success" size="mini"></u-tag>
					<u-tag v-else text="异常" type="error" size="mini"></u-tag>
				</view>
				<view slot="label" class="u-slot-label">
					<text class="u-cell-text">{{item.key}}</text>
				</view>
				<view slot="value" class="u-slot-value">
					<u-switch v-model="item.state" @change="change(item)" size="20"></u-switch>
				</view>
				<u-gap height="30" bgColor="#bbb"></u-gap>
			</u-cell>
		</u-cell-group>
		<u-loadmore v-if="isLoading" :status="status" :loading-text="loadingText" :loadmore-text="loadmoreText"
			:nomore-text="nomoreText" />
		<view class="add" @click="open">
			添加
		</view>
		<u-popup :show="show" :round="10" mode="center" @close="close" closeable>
			<view class="content">
				<u--form :model="form" ref="uForm" :rules="rules" labelPosition="top" labelWidth="auto">
					<u-form-item label="key" prop="key">
						<u-input v-model="form.key" placeholder="请输入有效key" />
					</u-form-item>
				</u--form>
				<u-button color="#6c6ceb" @click="submit">保存</u-button>
			</view>
		</u-popup>
	</view>
</template>

<script>
	import {
		getKey,
		upKey,
		addKey
	} from "@/uni_modules/chao-AI/js_sdk/index.js"
	export default {
		data() {
			return {
				isLoading: true,
				status: 'loadmore',
				loadingText: '努力加载中',
				loadmoreText: '轻轻上拉加载更多',
				nomoreText: '我是有底线的~',
				pageSize: 10,
				pageIndex: 1,
				show: false,
				KeyList: [],
				hasMore: false,
				form: {
					key: ''
				},
				rules: {
					key: [{
						type: 'string',
						required: true,
						message: '请输入key',
						trigger: ['blur', 'change']
					}]
				}
			}
		},
		async onLoad() {
			const data = {
				pageIndex: this.pageIndex,
				pageSize: this.pageSize
			}
			await this.handlegetKey(data)
		},
		onReachBottom() {
			if(!this.hasMore) return
			this.pageIndex = this.pageIndex + 1;
			const data = {
				pageIndex: this.pageIndex,
				pageSize: this.pageSize
			}
			this.handlegetKey(data)
		},
		methods: {
			open() {
				this.form.key = ''
				this.show = true
			},
			close() {
				this.show = false
			},
			submit() {
				this.$refs.uForm.validate().then(res => {
					addKey({
						id: uni.getAccountInfoSync().miniProgram.appId,
						key: this.form.key
					}).then(async (res) => {
						uni.$u.toast('添加成功')
						this.KeyList = []
						this.show = false
						this.pageIndex = 1
						const data = {
							pageIndex: this.pageIndex,
							pageSize: this.pageSize
						}
						await this.handlegetKey(data)
					})
				}).catch(errors => {
					uni.$u.toast('校验失败')
				})
			},
			async change(e) {
				const res = await upKey({
					id: e._id,
					state: e.state
				})
				if (res.updated === 1) {
					uni.$u.toast('修改成功')
				}
			},
			async handlegetKey({
				pageIndex,
				pageSize
			}) {
				this.status = 'loading';
				const res = await getKey({
					id: uni.getAccountInfoSync().miniProgram.appId,
					pageIndex,
					pageSize
				})
				const hasMore = res.data.length > this.pageSize - 1
				if (hasMore) {
					this.status = 'loadmore'
				} else {
					this.status = 'nomore'
				}
				this.hasMore = hasMore
				this.KeyList = this.KeyList.concat(res.data)
			}
		}
	}
</script>

<style lang="scss" scoped>
	.u-slot-title {
		display: flex;
		align-items: center;

		.u-cell-text {
			padding-right: 30rpx;
		}
	}

	.u-slot-label {
		font-size: 20rpx;
		padding: 20rpx 0;
	}

	.content {
		padding: 30rpx;
	}

	.add {
		animation: tiaobig 1.5s ease-in-out alternate infinite;
		background-color: #FFFFFF;
		position: fixed;
		bottom: 140rpx;
		right: 30rpx;
		border-radius: 120rpx;
		width: 90rpx;
		height: 90rpx;
		text-align: center;
		line-height: 90rpx;
		background-color: #6c6ceb;
		color: #fff;
		box-shadow: 0px 1px 8px #C8C8C8;

		image {
			width: 90rpx;
			height: 90rpx;
		}
	}

	@keyframes tiaobig {
		0% {
			transform: scale(0.92);
		}

		25% {
			transform: scale(1.08);
		}

		50% {
			transform: scale(0.96);
		}

		75% {
			transform: scale(1.06);
		}

		100% {
			transform: scale(0.94);
		}
	}
</style>
