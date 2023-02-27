<template>
	<view>
		<u-notice-bar bgColor='#6c6ceb' color="#fff" text="由于服务人数过多,如果出现异常建议重新尝试,AI对话已支持上下文连续对话欢迎体验" mode="closable">
		</u-notice-bar>
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
			<view class="flex_col">
				<view @click="handleopen">
					<u-icon name="grid" color="#000" size="28"></u-icon>
				</view>
				<view class="flex_grow">
					<input type="text" class="content" v-model="content" placeholder="请输入聊天内容"
						placeholder-style="color:#DDD;" :cursor-spacing="6">
				</view>
				<button class="send" @tap="send">发送</button>
			</view>
		</view>
		<u-popup :show="show" :round="10" mode="bottom" @close="close" closeable>
			<view class="UPOP">
				<view class="title ">我的菜单栏</view>
				<view class="margin">
					<view class="userInfo">
						<view class="info">
							<view class="vip" v-if="isday">会员</view>
							<view class="name " v-else>普通用户</view>
							<view class="id ">{{'ID: '+ openid}}</view>
						</view>
					</view>
					<view class="stat ">
						<view class="have ">{{'拥有额度：'+isnum+'次'}}</view>
						<view class="free ">{{'每日免费额度：'+freePerDay+'次'}}</view>
					</view>
					<view class="invite ">
						<view class="info ">
							<view class="inviteTitle ">邀请新的小伙伴来体验</view>
							<view class="award ">奖励<text class="span ">{{invite}}</text>次/人，每天最多<text
									class="span ">{{10}}</text>人</view>
							<view class="hint ">提示：点击右上角<text class="more ">···</text>可分享朋友圈邀请</view>
						</view>
						<view class="btn ">邀请朋友<button openType="share">邀请</button>
						</view>
					</view>
					<u-line></u-line>
					<view class="videoAd">
						<view class="info">
							<view class="videoAdTitle ">看视频广告攒次数</view>
							<view class="award ">奖励<text class="span ">{{advertisement}}</text>次/个</view>
						</view>
						<view class="btn" @click="handleAd">观看视频</view>
					</view>
				</view>
				<u-button open-type='contact' color="#6c6ceb" :plain="true" text="咨询客服(意见反馈)"></u-button>
			</view>
		</u-popup>
	</view>
</template>

<script>
	import {
		checkText,
		getSend,
		AddCount
	} from "@/uni_modules/chao-AI/js_sdk/index.js"
	import {
		handleConfig,
		handleTime
	} from "@/uni_modules/chao-AI/js_sdk/common.js"
	// 在页面中定义激励视频广告
	let videoAd = null
	export default {
		data() {
			return {
				talkList: [],
				content: '',
				show: false,
				isday: 0,
				isnum: 5,
				invite: 0,
				freePerDay: 0,
				advertisement: 0,
				openid: '',
				obj: {},
				id: ''
			}
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
		onShareAppMessage: function(options) {
			const that = this;
			// 设置菜单中的转发按钮触发转发事件时的转发内容
			const shareObj = {
				title: '一键AI问答',
				imageUrl: '/static/wenda.png', //自定义图片路径，可以是本地文件路径、代码包文件路径或者网络图片路径，支持PNG及JPG，不传入 imageUrl 则使用默认截图。显示图片长宽比是 5:4
				path: '/pages/main/duihua/duihua?id=' + that.openid,
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
				path: '/pages/main/duihua/duihua?id=' + that.openid,
			};
			// 返回shareObj
			return shareObj;
		},
		mounted() {
			this.$nextTick(() => {
				this.getHistoryMsg();
			});
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
			close() {
				this.show = false
			},
			handleopen() {
				this.show = true
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
				const taht = this
				if (!taht.content) {
					uni.showToast({
						title: '请输入有效的内容',
						icon: 'none'
					})
					return;
				}
				if (taht.isnum < 1) {
					uni.showToast({
						title: '免费额度已用完,请点击左下角获取次数',
						duration: 3000,
						mask: true,
						icon: 'none'
					});
					return
				}
				if (taht.isday === 0) {
					taht.isnum = taht.isnum - 1
					uni.setStorageSync('config', {
						...taht.obj,
						num: taht.isnum
					});
				}
				uni.showLoading({
					title: '文本安全检测中',
					mask: true
				})
				const resa = await checkText({
					params: {
						prompt: taht.content,
						openid: taht.openid,
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
					uni.showLoading({
						title: '正在发送',
						mask: true
					})
					let query = ''
					taht.talkList.forEach(item => {
						if (item.type === 1) {
							query += `\nQ: ${item.content}\n`
						} else {
							query += `A: ${item.content}。 <|endoftext|>\n`
						}
					})
					query += `Q: ${taht.content}  + "\nA: `
					const res = await getSend({
						prompt: query,
						openid: taht.openid,
						id: uni.getAccountInfoSync().miniProgram.appId,
					})
					// 将当前发送信息 添加到消息列表。
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
	}
</script>

<style lang="scss">
	@import "duihua.scss"
</style>
