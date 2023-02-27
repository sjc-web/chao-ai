<template>
	<view>
		</u-notice-bar>
		<u-cell-group>
			<u-cell v-for="(item, index) in KeyList" :key="item._id">
				<view slot="title" class="u-slot-title">
					<text class="u-cell-text">{{$u.timeFormat(item.create_data, 'yyyy-mm-dd')}}</text>
					<u-tag v-if="item.day === 1" text="会员" type="success" size="mini"></u-tag>
					<u-tag v-else text="普通用户" type="error" size="mini"></u-tag>
				</view>
				<view slot="label" class="u-slot-label">
					<view>免费额度：{{item.num}}</view>
					<view>ID：{{item.openid}}</view>
				</view>
				<view slot="value" class="u-slot-value">
					<u-tag text="编辑" plain type="success" size="mini" @click="change(item)"></u-tag>
				</view>
				<u-gap height="30" bgColor="#bbb"></u-gap>
			</u-cell>
		</u-cell-group>
		<u-loadmore v-if="isLoading" :status="status" :loading-text="loadingText" :loadmore-text="loadmoreText"
			:nomore-text="nomoreText" />
		<u-popup :show="show" :round="10" mode="center" @close="close" closeable>
			<view class="content">
				<u--form :model="form" ref="uForm" :rules="rules" labelPosition="top" labelWidth="auto">
					<u-form-item label="是否开通会员(0/1)" prop="day">
						<u-input type="number" v-model="form.day" placeholder="请输入1是会员" />
					</u-form-item>
					<u-form-item label="使用额度" prop="num">
						<u-input type="number" v-model="form.num" placeholder="请输入" />
					</u-form-item>
				</u--form>
				<u-button color="#6c6ceb" @click="submit">保存</u-button>
			</view>
		</u-popup>
	</view>
</template>

<script>
	import {
		getUser,
		upUser,
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
					day: 0,
					num: 0
				},
				rules: {
					day: [{
						type: 'number',
						required: true,
						message: '请输入',
						trigger: ['blur', 'change']
					}],
					num: [{
						type: 'number',
						required: true,
						message: '请输入',
						trigger: ['blur', 'change']
					}],
				}
			}
		},
		async onLoad() {
			const data = {
				pageIndex: this.pageIndex,
				pageSize: this.pageSize
			}
			await this.handlegetUser(data)
		},
		onReachBottom() {
			if (!this.hasMore) return
			this.pageIndex = this.pageIndex + 1;
			const data = {
				pageIndex: this.pageIndex,
				pageSize: this.pageSize
			}
			this.handlegetUser(data)
		},
		methods: {
			close() {
				this.show = false
			},
			submit() {
				this.$refs.uForm.validate().then(res => {
					upUser(this.form).then(async (res) => {
						uni.$u.toast('编辑成功')
						this.KeyList = []
						this.show = false
						this.pageIndex = 1
						const data = {
							pageIndex: this.pageIndex,
							pageSize: this.pageSize
						}
						await this.handlegetUser(data)
					})
				}).catch(errors => {
					uni.$u.toast('校验失败')
				})
			},
			async change(e) {
				this.form = e
				this.show = true

			},
			async handlegetUser({
				pageIndex,
				pageSize
			}) {
				this.status = 'loading';
				const res = await getUser({
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
		font-size: 24rpx;
		padding: 20rpx 0;
		line-height: 50rpx;
	}

	.content {
		padding: 30rpx;
	}
</style>
