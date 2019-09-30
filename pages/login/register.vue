<template>
	<view class="register">

		<view class="content">
			<!-- 头部logo -->
			<view class="header">
				<image :src="logoImage"></image>
			</view>
			<!-- 主体 -->
			<view class="main">
				<wInput v-model="registerForm.opername" type="text" placeholder="登录账号" maxlength="20"></wInput>
				<wInput v-model="registerForm.password" type="password" maxlength="20" placeholder="登录密码" isShowPass></wInput>
				<wInput v-model="registerForm.passwordTwo" type="password" maxlength="20" placeholder="密码确认" isShowPass></wInput>
				<wInput v-model="registerForm.phone" type="text" maxlength="11" placeholder="手机号"></wInput>
				<wInput v-model="registerForm.email" type="text" placeholder="邮箱"></wInput>
				<wInput v-model="registerForm.emailva" type="number" maxlength="6" placeholder="验证码" isShowCode ref="runCode"
				 @setCode="getVerificationCode()"></wInput>
			</view>
			<wButton text="注 册" :rotate="isRotate" @click.native="register()"></wButton>
			<!-- 底部信息 -->
			<view class="footer">
				<text @tap="isShowAgree" class="cuIcon" :class="showAgree?'cuIcon-radiobox':'cuIcon-round'">同意</text>
				<!-- 协议地址 -->
				<navigator url="" open-type="navigate">《协议》</navigator>
			</view>
		</view>
	</view>
</template>

<script>
	var _this;
	import wInput from '../../components/watch-login/watch-input.vue' //input
	import wButton from '../../components/watch-login/watch-button.vue' //button

	export default {
		data() {
			return {
				//logo图片 base64
				logoImage: 'data:image/jpeg;base64,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',
				registerForm: {
					opername: '', // 用户/电话
					password: '',
					passwordTwo: '',
					phone: '',
					email: '',
					emailva: '',
					babybirth: new Date(),
					type: 1,
				},
				passData: '', //密码
				verCode: "", //验证码
				showAgree: true, //协议是否选择
				isRotate: false, //是否加载旋转
			}
		},
		components: {
			wInput,
			wButton,
		},
		mounted() {
			_this = this;
		},
		methods: {
			isShowAgree() {
				//是否选择协议
				_this.showAgree = !_this.showAgree;
			},
			getVerificationCode() {
				//获取验证码
				if (_this.registerForm.email.length == '') {
					uni.showToast({
						icon: 'none',
						position: 'bottom',
						title: '邮箱不能为空'
					});
					return false;
				}
				var url = getApp().globalData.url;
				//var url = 'https://203.195.177.99';
				var reg = new RegExp(
					/^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
				);
				if (!reg.test(_this.registerForm.email)) {
					uni.showToast({
						icon: 'none',
						position: 'bottom',
						title: '邮箱格式错误'
					});
					return;
				}
				wx.request({
					url: url + '/wxxcx/login/default.do?method=veriyzm&email=' + _this.registerForm.email,
					data: null,
					method: "GET",
					header: {
						'content-type': 'application/x-www-form-urlencoded;charset=utf-8', // 默认值
						'Cookie': wx.getStorageSync("sessionId")
					},
					success(res) {
						wx.setStorageSync("sessionId", 'JSESSIONID=' + res.data.xionghaizhiS);
						if (res.data.result != 'true') {
							uni.showToast({
								icon: 'none',
								position: 'bottom',
								title: res.data.msg,
							});
							return;
						}
						_this.$refs.runCode.$emit('runCode'); //触发倒计时（一般用于请求成功验证码后调用）
						uni.showToast({
							icon: 'none',
							position: 'bottom',
							title: '验证码发送成功',
						});
					},
					fail: function() {
						uni.showToast({
							icon: 'none',
							position: 'bottom',
							title: '服务问题，稍候重试',
						});
					}
				})

				/* 		setTimeout(function() {
							_this.$refs.runCode.$emit('runCode', 0); //假装模拟下需要 终止倒计时
							uni.showToast({
								icon: 'none',
								position: 'bottom',
								title: '模拟倒计时终止'
							});
						}, 3000) */
			},
			checkPass(e, r) {
				if (e === r) {
					return true;
				}
				return false;
			},
			//验证用户名：用户名，4到16位（字母，数字，下划线，减号）
			verifiUserName(e) {
				var reg = new RegExp('^[a-zA-Z0-9_-]{4,16}$');
				return reg.test(e); // 得到的值为布尔型
			},
			showM(e) {
				uni.showToast({
					icon: 'none',
					position: 'bottom',
					title: e
				});
			},
			//验证密码：密码必须6到12位，且不能出现空格
			verifiPassWord(e) {
				var vaTrim = e.trim();
				if (e.length >= 6 && e.length <= 12 && e.length == vaTrim.length) {
					return true;
				}
				return false; // 得到的值为布尔型
			},
			register() {
				//注册
				if (this.isRotate) {
					//判断是否加载中，避免重复点击请求
					return false;
				}
				if (this.showAgree == false) {
					_this.showM("请先同意《协议》");
					return false;
				}
				if (!_this.checkPass(_this.registerForm.password, _this.registerForm.passwordTwo)) {
					_this.showM("两次密码不同");
					return;
				}
				if (!(_this.verifiUserName(_this.registerForm.opername))) {
					_this.showM("用户名，4到16位（字母，数字，下划线，减号）");
					return;
				}
				if (!(_this.verifiPassWord(_this.registerForm.password))) {
					_this.showM('密码必须6到12位，且不能出现空格）');
					return;
				}
				if (_this.registerForm.phone.length != 11) {
					_this.showM('手机号不正确');
					return false;
				}
				if (_this.registerForm.email == '') {
					_this.showM('邮箱不能为空');
					return false;
				}
				if (_this.registerForm.email == '') {
					_this.showM('邮箱不能为空');
					return false;
				}
				var reg = new RegExp(
					/^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
				);
				if (!reg.test(_this.registerForm.email)) {
					_this.showM('邮箱格式错误');
					return false;
				}
				if (_this.registerForm.emailva == '') {
					_this.showM('验证码不能为空');
					return false;
				}
				if (_this.registerForm.emailva.length != 6) {
					_this.showM('验证码不正确');
					return false;
				}
				_this.isRotate = true
				var url = getApp().globalData.url;
				//var url = 'https://203.195.177.99';
				wx.request({
					url: url + '/wxxcx/login/default.do?method=verificationOperName&cx_opername=' + _this.registerForm.opername +
						'&cx_email=' + _this.registerForm.email,
					data: null,
					method: "GET",
					header: {
						'content-type': 'application/x-www-form-urlencoded;charset=utf-8', // 默认值
						'Cookie': wx.getStorageSync("sessionId")
					},
					success(res) {
						if (res.data.result != 'true') {
							_this.showM(res.data.msg);
							_this.isRotate = false
							return;
						}
						wx.request({
							url: url + '/wxxcx/login/default.do?method=register',
							data: _this.registerForm,
							method: "POST",
							header: {
								'content-type': 'application/x-www-form-urlencoded;charset=utf-8', // 默认值
								'Cookie': wx.getStorageSync("sessionId")
							},
							success(res) {
								if (res.data.result != 'true') {
									_this.showM(res.data.msg);
									_this.isRotate = false
									return;
								}
								_this.showM('注册成功');
								_this.isRotate = false
								//注册成功后跳转登录页面
								setTimeout(function() {
									wx.redirectTo({
										url: '../login/login',
									})
								}, 2500)
							},
							fail: function() {
								_this.showM('服务问题，稍候重试');
								return;
							}
						})
					},
					fail: function() {
						_this.showM('服务问题，稍候重试');
						return;
					}
				})
				/* 	setTimeout(function() {
						_this.isRotate = false
					}, 3000) */
			}
		}
	}
</script>

<style>
	@import url("../../components/watch-login/css/icon.css");
	@import url("./css/main.css");
</style>
