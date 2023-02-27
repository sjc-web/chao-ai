<template>
	<view class="container">
		<view class="admin-top">
			<view class="nums">数据明细</view>
		</view>
		<!-- <view class="bg-white border-bottom headers">
			<view class="title">数据统计</view>
		</view> -->
		<u-gap height="15" bg-color="#F5F6F8"></u-gap>
		<view class="bg-white border-bottom headers">
			<view class="title">系统菜单</view>
		</view>
		<u-grid :border="false" col="3" @click="click">
			<u-grid-item v-for="(listItem,listIndex) in list" :key="listIndex">
				<u-icon :customStyle="{paddingTop:20+'rpx'}" :name="listItem.name" :size="22"></u-icon>
				<text class="grid-text">{{listItem.title}}</text>
			</u-grid-item>
		</u-grid>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [{
						name: 'grid-fill',
						title: '应用配置',
						path: '/pages/admin/application/application'
					},
					{
						name: 'edit-pen-fill',
						title: 'Key管理',
						path: '/pages/admin/keymanagement/keymanagement'
					},
					{
						name: 'account-fill',
						title: '用户管理',
						path: '/pages/admin/usermanagement/usermanagement'
					}
				]
			}
		},
		methods: {
			click(name) {
				uni.navigateTo({
					url: this.list[name].path
				})
			}
		}
	}
</script>

<style lang="scss">
	.admin-top {
		display: flex;
		flex-direction: row;
		align-items: center;
		width: 100vw;
		height: 145px;
		margin-top: -133px;
		padding-top: 133px;
		background-image: linear-gradient(100deg, #6c6ceb, #6c6ceb);
		justify-content: space-around;

		.nums {
			color: #fff;
			font-size: 40rpx;
			font-weight: 550;
		}
	}

	.headers {
		padding: 30rpx;
	}

	.border-bottom {
		border-bottom: 1rpx solid #F3F3F3;
	}

	.bg-white {
		background-color: #FFFFFF;
	}

	.title {
		border-left: 6rpx solid #333333;
		padding-left: 15rpx;
		line-height: 28rpx;
		color: #333333;
		font-weight: 550;
	}

	.grid-text {
		font-size: 14px;
		color: #909399;
		padding: 10rpx 0 20rpx 0rpx;
		/* #ifndef APP-PLUS */
		box-sizing: border-box;
		/* #endif */
	}
</style>
