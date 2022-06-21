<template>
	<view>
		<uni-forms ref="form">
			<uni-forms-item label="链接地址" name="url">
				<uni-easyinput type="text" v-model="url" />
			</uni-forms-item>
			<uni-forms-item label="信息查询" name="message">
				<uni-easyinput v-model="message" type="text" placeholder="服务器-服务名 eg. hk03-mysql" />
			</uni-forms-item>
		</uni-forms>
		<span v-if="socket_status">socket 已经连接</span>
		<span v-else>socket 没有连接</span>
		<button @click="socket_start()">start socket</button>
		<button @click="socket_close()">close socket</button>
		<button @click="socket_send()">信息查询</button>
		<ul id="message_box">
			<li v-for="(v,k) in message_list">
				{{ v }}
			</li>
		</ul>
		<button @click="clear()">clear screen</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				url: "ws://47.108.88.145:8080/notify",
				message: "",
				socket_status: false,
				timer: false,
				message_list: []
			}
		},
		onLoad() {
			let that = this
			uni.onSocketOpen(function(res) {
				that.socket_status = true
				console.log('WebSocket连接已打开！');
			});
			uni.onSocketError(function(res) {
				uni.showToast({
					title: "WebSocket连接打开失败，请检查！",
					icon: "error"
				})
				console.log('WebSocket连接打开失败，请检查！');
			});
			uni.onSocketClose(function(res) {
				that.socket_status = false
				console.log('WebSocket 已关闭！');
				clearInterval(that.timer)
			});
			uni.onSocketMessage(function(res) {
				var info = JSON.parse(decodeURIComponent(res.data))
				var platform = uni.getSystemInfoSync().platform
				if (info['method'] == 'show') {
					for (var i in info['data']) {
						var server_arr = i.split('-')
						var done_info = '[' + info['data'][i] + '] ' + server_arr["1"] + ' ' + server_arr["0"]
						if (platform == 'android' && info['is_push']) {
							let option = {
								cover: false,
								sound: "system",
								title: "some thing " + info['data'][i] + "!"
							}
							plus.push.createMessage(done_info, "", option)
							
						}
						that.message_list.unshift(done_info)
					}

					that.message_list = that.message_list.splice(0, 10);
				} else {
					console.log(info)
				}
			});
		},
		methods: {
			socket_ping() {
				this.timer = setInterval(function() {
					uni.sendSocketMessage({
						data: '{"cmd":"notify.ping"}'
					});
				}, 30000)
			},
			socket_start() {
				this.$refs.form.validate().then(res => {
					uni.connectSocket({
						url: this.url,
						header: {
							'content-type': 'application/json'
						},
						protocols: ['protocol1'],
						method: 'GET'
					});
					this.socket_ping()
				}).catch(err => {
					uni.showToast({
						title: err,
						icon: "error"
					})
				})


			},
			socket_close() {
				uni.closeSocket();
			},
			socket_send() {
				if (this.socket_status) {
					uni.sendSocketMessage({
						data: '{"cmd":"notify.index","data":"' + this.message + '"}'
					});
				} else {
					uni.showToast({
						title: "socket 没有连接",
						icon: "error"
					})
				}
			},
			clear() {
				this.message_list = []
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
