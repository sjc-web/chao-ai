<template>
	<view class="container">
		<u--form :model="form" ref="uForm" :rules="rules" labelPosition="top" labelWidth="auto">
			<u-form-item label="appid" prop="appid">
				<u-input disabled v-model="form.appid" placeholder="请输入小程序appid" />
			</u-form-item>
			<u-form-item label="secret" prop="secret">
				<u-input v-model="form.secret" placeholder="请输入小程序secret" />
			</u-form-item>
			<u-form-item label="每天免费额度" prop="freePerDay">
				<u-input type="number" v-model="form.freePerDay" placeholder="请输入" />
			</u-form-item>
			<u-form-item label="激励奖励次/个" prop="advertisement">
				<u-input type="number" v-model="form.advertisement" placeholder="请输入" />
			</u-form-item>
			<u-form-item label="邀请奖励次/人" prop="invite">
				<u-input type="number" v-model="form.invite" placeholder="请输入" />
			</u-form-item>
			<u-form-item label="签名" prop="sign">
				<u-input v-model="form.sign" placeholder="请输入签名" />
			</u-form-item>
		</u--form>
		<u-button color="#6c6ceb" @click="submit">保存</u-button>
	</view>
</template>

<script>
	import {
		amConfig,
		upConfig
	} from "@/uni_modules/chao-AI/js_sdk/index.js"
	export default {
		data() {
			return {
				form: {
					appid: '',
					secret: '',
					sign: '',
					advertisement: 0,
					freePerDay: 0,
					invite: 0
				},
				rules: {
					appid: [{
						type: 'string',
						required: true,
						message: '请输入小程序appid',
						trigger: ['blur', 'change']
					}],
					secret: [{
						type: 'string',
						required: true,
						message: '请输入小程序secret',
						trigger: ['blur', 'change']
					}],
					advertisement: [{
						type: 'number',
						required: true,
						message: '请输入',
						trigger: ['blur', 'change']
					}],
					freePerDay: [{
						type: 'number',
						required: true,
						message: '请输入',
						trigger: ['blur', 'change']
					}],
					invite: [{
						type: 'number',
						required: true,
						message: '请输入',
						trigger: ['blur', 'change']
					}],
					sign: [{
						type: 'string',
						required: true,
						message: '请输入小程序签名',
						trigger: ['blur', 'change']
					}]
				}
			};
		},
		async onLoad() {
			uni.showLoading({
				title: '加载中',
				mask: true
			})
			const res = await amConfig({
				id: uni.getAccountInfoSync().miniProgram.appId,
			})
			if (res.data.length > 0) {
				this.form = res.data[0]
			}
			uni.hideLoading()
		},
		methods: {
			submit() {
				this.$refs.uForm.validate().then(res => {
					upConfig(this.form).then(res => {
						uni.$u.toast('保存成功')
					})
				}).catch(errors => {
					uni.$u.toast('校验失败')
				})
			}
		},
	};
</script>
<style lang="scss" scoped>
	.container {
		padding: 30rpx;
	}
</style>
