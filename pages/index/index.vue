<template>
	<view>
		<uni-forms ref="form">
			<uni-forms-item label="链接地址" name="url">
				<uni-easyinput type="text" v-model="url" />
			</uni-forms-item>
			<uni-forms-item label="发送信息" name="message">
				<input class="input" v-model="message" type="text" />
			</uni-forms-item>
		</uni-forms>
		<span v-if="socket_status">socket 已经连接</span>
		<span v-else>socket 没有连接</span>
		<button @click="socket_start()">start socket</button>
		<button @click="socket_close()">close socket</button>
		<button @click="socket_send()">send data</button>
		<ul id="message_box">
		  <li v-for="item in message_list">
		    {{ item }}
		  </li>
		</ul>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				url: "http://192.168.31.79:3000",
				message: "",
				socket_status: false,
				message_list : []
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
			});
			uni.onSocketMessage(function (res) {
				that.message_list.unshift(res.data)
				that.message_list = that.message_list.splice(0,10);
				console.log(that.message_list)
			});
		},
		methods: {

			socket_start() {
				this.$refs.form.validate().then(res => {
					uni.connectSocket({
						url: this.url,
						data() {
							return {
								x: '',
								y: ''
							};
						},
						header: {
							'content-type': 'application/json'
						},
						protocols: ['protocol1'],
						method: 'GET'
					});
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
					if (this.message) {
						uni.sendSocketMessage({
							data: this.message
						});
					} else {
						uni.showToast({
							title: "message 没有内容",
							icon: "error"
						})
					}
				} else {
					uni.showToast({
						title: "socket 没有连接",
						icon: "error"
					})
				}
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
