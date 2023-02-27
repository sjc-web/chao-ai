<template>
	<view>
		<view class="box-1" id="list-box">
			<view class="talk-list">
				<view v-for="(item,index) in talkList" @click="handleCopy(item.content)" :key="index"
					:id="`msg-${item.id}`">
					<view class="item flex_col" :class=" item.type == 1 ? 'push':'pull' ">
						<image :src="item.pic" mode="aspectFill" class="pic"></image>
						<view class="content">{{item.content}}</view>
					</view>
				</view>
			</view>
		</view>
		<view class="box-2">
			<view class="flex_col" v-if="key">
				<view class="flex_grow">
					<input type="text" class="content" v-model="content" placeholder="请输入聊天内容"
						placeholder-style="color:#DDD;" :cursor-spacing="6">
				</view>
				<button class="send" @tap="send">发送</button>
			</view>
			<view class="flex_col" v-else>
				<view class="flex_grow">
					<input type="text" class="content" v-model="keys" placeholder="请输入key,sk-开头,首页查看注册攻略"
						placeholder-style="color:#DDD;" :cursor-spacing="6">
				</view>
				<button class="send" @tap="gotu">登录</button>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		checkText,
		mySend,
	} from "@/uni_modules/chao-AI/js_sdk/index.js"
	import {
		handleConfig
	} from "@/uni_modules/chao-AI/js_sdk/common.js"
	export default {
		data() {
			return {
				talkList: [],
				content: '',
				key: '',
				keys: '',
				openid: ''
			}
		},
		onLoad() {
			const value = uni.getStorageSync('config');
			if (value) {
				const {
					key,
					openid
				} = value
				this.key = key
				this.openid = openid
				this.obj = value
				return
			}
			const that = this
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
		mounted() {
			this.$nextTick(() => {
				this.getHistoryMsg();
			});
		},
		methods: {
			async handlefig(code) {
				const res = await handleConfig(code)
				const {
					key,
					openid
				} = res
				this.key = key
				this.openid = openid
				this.obj = res
				uni.hideLoading()
				uni.setStorageSync('config', res);
			},
			async gotu() {
				uni.showLoading({
					title: '登录中',
					mask: true
				})
				const taht = this
				await mySend({
					key: taht.keys,
					prompt: '你好',
					openid: taht.openid || new Date().getTime() + '',
				})
				uni.hideLoading();
				uni.showToast({
					title: '登录成功，开启AI之旅',
					duration: 2000,
					mask: true,
					icon: 'none'
				});
				uni.setStorageSync('config', {
					...taht.obj,
					key: taht.keys
				});
				taht.key = taht.keys
			},
			handleCopy(message) {
				const that = this
				uni.setClipboardData({
					data: message,
					success: function() {}
				})
			},
			// 获取历史消息
			getHistoryMsg() {
				let get = async () => {
					let data = [{
						"id": '1', // 消息的ID
						"content": `GPT智能AI为您服务：\n1. 知乎百度答题、做作业题目\n2. 写代码、写文案、写论文，写小说\n3. 文案润色、翻译、写诗作词\n4. 扮演面试官、扮演书籍电影角色\n例1：写一篇工作日报我是行政\n例2：帮我写作业，题目是xxx\n例3：把下文翻译成英文：xxx\n例4：写一出能活的短视频剧本\n例5：英文写物理相关的论文\n例6：用王小波的风格写篇情书\n
或者可以问我其他问题\n越完整的描述，我越精确`, // 消息内容
						"type": 0, // 此为消息类别，设 1 为发出去的消息，0 为收到对方的消息,
						"pic": "https://wx.qlogo.cn/mmhead/Q3auHgzwzM55kOdw21icYsGmEojHmk3XMqRLWIZPkZJ6uBLvqdf6SxA/0"
					}];

					const selector = `#msg-${data[data.length-1].id}`;
					// 将获取到的消息数据合并到消息数组中
					this.talkList = [...data, ...this.talkList];
					// 数据挂载后执行，不懂的请自行阅读 Vue.js 文档对 Vue.nextTick 函数说明。
					this.$nextTick(() => {
						// 设置当前滚动的位置
						this.setPageScrollTo(selector);
					})
				}
				get();
			},
			// 设置页面滚动位置
			setPageScrollTo(selector) {
				let view = uni.createSelectorQuery().in(this).select(selector);
				view.boundingClientRect((res) => {
					uni.pageScrollTo({
						scrollTop: res.top - 30, // -30 为多显示出大半个消息的高度，示意上面还有信息。
						duration: 0
					});
				}).exec();
			},
			// 发送信息
			async send() {
				if (!this.content) {
					uni.showToast({
						title: '请输入有效的内容',
						icon: 'none'
					})
					return;
				}
				uni.showLoading({
					title: '正在发送',
					mask: true
				})
				const taht = this
				let query = ''
				taht.talkList.forEach(item => {
					if (item.type === 1) {
						query += `\nQ: ${item.content}\n`
					} else {
						query += `A: ${item.content}。 <|endoftext|>\n`
					}
				})
				query += `Q: ${taht.content}  + "\nA: `
				const res = await mySend({
					key: taht.key,
					prompt: query,
					openid: taht.openid || new Date().getTime() + '',
				})
				taht.talkList.push({
					"id": new Date().getTime(),
					"content": taht.content,
					"type": 1,
					"pic": "https://wx.qlogo.cn/mmhead/Q3auHgzwzM55kOdw21icYsGmEojHmk3XMqRLWIZPkZJ6uBLvqdf6SxA/0"
				}, {
					"id": new Date().getTime(),
					"content": res,
					"type": 0,
					"pic": "https://wx.qlogo.cn/mmhead/Q3auHgzwzM55kOdw21icYsGmEojHmk3XMqRLWIZPkZJ6uBLvqdf6SxA/0"
				});
				uni.hideLoading()
				taht.$nextTick(() => {
					// 清空内容框中的内容
					taht.content = '';
					uni.pageScrollTo({
						scrollTop: 999999, // 设置一个超大值，以保证滚动条滚动到底部
						duration: 0
					});
				})
			}
		}
	}
</script>

<style lang="scss">
	@import "../duihua/duihua.scss"
</style>
