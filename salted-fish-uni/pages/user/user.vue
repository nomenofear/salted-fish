<template>
	<!--pages/user/user.wxml-->
	<view class="container">
		<view class="userinfo" v-if="isAuthInfo" @tap="toUserInfoPage">
			<view class="userinfo-con">
				<view class="userinfo-avatar">
					<!-- <open-data type="userAvatarUrl"></open-data> -->
					<image :src="loginResult.data.avatarImage?serverUrl+loginResult.data.avatarImage:'../../static/images/icon/head04.png'"></image>
				</view>
				<view class="userinfo-name">
					<view>{{loginResult.data.username ? loginResult.data.username : "用户昵称"}}</view>
					<!-- <open-data type="userNickName"></open-data> -->
				</view>
			</view>
		</view>

		<view class="userinfo-none" v-if="!isAuthInfo">
			<view class="default-pic">
				<image src="../../static/images/icon/head04.png"></image>
			</view>
			<view class="none-login">
				<button class="unlogin" @tap="toLogin">未登录</button>
				<button class="click-login" @tap="toLogin">点击账号登录</button>
			</view>
		</view>
		<!-- end 用户信息 -->

		<view class="list-cont">

			<!-- 订单状态 -->
			<view class="total-order">
				<view class="order-tit">
					<text style="font-weight:bold">我的订单</text>
					<view class="checkmore" @tap="toOrderListPage" data-sts="0">
						<text>查看全部 </text>
						<text class="arrowhead"></text>
					</view>
				</view>
				<view class="procedure">
					<view class="items" @tap="toOrderListPage" data-sts="1">
						<image src="/static/images/icon/toPay.png"></image>
						<text>待支付</text>
						<text class="num-badge" v-if="orderAmount.unPay>0">{{orderAmount.unPay}}</text>
					</view>
					<view class="items" @tap="toOrderListPage" data-sts="2">
						<image src="/static/images/icon/toDelivery.png"></image>
						<text>待发货</text>
						<text class="num-badge" v-if="orderAmount.payed>0">{{orderAmount.payed}}</text>
					</view>
					<view class="items" @tap="toOrderListPage" data-sts="3">
						<image src="/static/images/icon/toTake.png"></image>
						<text>待收货</text>
						<text class="num-badge" v-if="orderAmount.consignment>0">{{orderAmount.consignment}}</text>
					</view>
					<view class="items" @tap="toOrderListPage" data-sts="5">
						<image src="/static/images/icon/toComment.png"></image>
						<text>已完成</text>
					</view>
				</view>
			</view>
			<!--end 订单状态 -->

			<view class="prod-col">
				<view class="col-item" @tap="myCollectionHandle">
					<view class="num">{{collectionCount}}</view>
					<view class="tit">我的收藏</view>
				</view>
				<view class="col-item" @tap="toOrderListPage" data-sts="0">
					<view class="num">{{orderAmount}}</view>
					<view class="tit">我的订单</view>
				</view>
			</view>

			<view class="my-menu">
				<view class="memu-item" @tap="toDistCenter">
					<view class="i-name">
						<image src="/static/images/icon/promotion.png"></image>
						<text>我发布的</text>
					</view>
					<view class="arrowhead"></view>
				</view>
				<view class="memu-item" @tap="toCouponCenter">
					<view class="i-name">
						<image src="/static/images/icon/getCoupon.png"></image>
						<text>我卖出的</text>
					</view>
					<view class="arrowhead"></view>
				</view>
				<!-- <view class="memu-item" @tap="toMyCouponPage">
					<view class="i-name">
						<image src="/static/images/icon/myCoupon.png"></image>
						<text>我的优惠券</text>
					</view>
					<view class="arrowhead"></view>
				</view> -->
				<view class="memu-item" @tap="toAddressList">
					<view class="i-name">
						<image src="/static/images/icon/myAddr.png"></image>
						<text>收货地址</text>
					</view>
					<view class="arrowhead"></view>
				</view>
			</view>
			<!--end 列表项 -->

			<view class="log-out" @tap="logout" v-if="isAuthInfo">
				<view class="log-out-n">
					<text>退出登录</text>
				</view>
			</view>
		</view>

	</view>
</template>

<script>
	// pages/user/user.js
	var http = require("../../utils/http.js");
	var util = require("../../utils/util.js");
	var config = require("../../utils/config.js");

	export default {
		data() {
			return {
				orderAmount: 0,
				sts: '',
				collectionCount: 0,
				isAuthInfo: false,
				loginResult: {
					    data: {
							code: 0,
							data: {
								avatarImage: "",
								createTime: '',
								id: '',
								password: "",
								phone: "",
								username: "",
							},
							msg: ""
						},
					    type: ""
				},
				serverUrl: config.domain
			};
		},
		components: {},
		props: {},

		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad: function(options) {},

		/**
		 * 生命周期函数--监听页面初次渲染完成
		 */
		onReady: function() {},

		/**
		 * 生命周期函数--监听页面显示
		 */
		onShow: function() {
			//加载订单数字
			var ths = this; // var status = ths.data.status
			ths.setData({
				loginResult: uni.getStorageSync("loginResult"),
				// isAuthInfo: Boolean(wx.getStorageSync('loginResult').userId),
			});
			if (ths.loginResult) {
				ths.setData({
					isAuthInfo: true
				})
			} else {
				ths.setData({
					isAuthInfo: false
				})
			}
			if (ths.isAuthInfo) {
				uni.showLoading();
				var params = {
					url: "/order/count",
					method: "GET",
					needToken: true,
					data: {},
					callBack: function(res) {
						uni.hideLoading();
						ths.setData({
							orderAmount: res
						});
					}
				};
				http.request(params);
				this.showCollectionCount();
			}
		},

		/**
		 * 生命周期函数--监听页面隐藏
		 */
		onHide: function() {},

		/**
		 * 生命周期函数--监听页面卸载
		 */
		onUnload: function() {},

		/**
		 * 页面相关事件处理函数--监听用户下拉动作
		 */
		onPullDownRefresh: function() {},

		/**
		 * 页面上拉触底事件的处理函数
		 */
		onReachBottom: function() {},

		/**
		 * 用户点击右上角分享
		 */
		onShareAppMessage: function() {},
		methods: {
			toDistCenter: function() {
				uni.navigateTo({
					url: '/pages/MyReleased/MyReleased'
				});
			},
			toCouponCenter: function() {
				uni.navigateTo({
					url: '/pages/MySell/MySell'
				});
			},
			toMyCouponPage: function() {
				uni.showToast({
					icon: "none",
					title: '该功能未开发'
				});
			},
			toAddressList: function() {
				uni.navigateTo({
					url: '/pages/delivery-address/delivery-address'
				});
			},
			// 跳转绑定手机号
			toBindingPhone: function() {
				uni.navigateTo({
					url: '/pages/binding-phone/binding-phone'
				});
			},
			toOrderListPage: function(e) {
				var sts = e.currentTarget.dataset.sts;
				uni.navigateTo({
					url: '/pages/orderList/orderList?sts=' + sts
				});
			},

			/**
			 * 查询所有的收藏量
			 */
			showCollectionCount: function() {
				var ths = this;
				uni.showLoading();
				var params = {
					url: "/flower/favorite/count",
					method: "GET",
					needToken: true,
					data: {},
					callBack: function(res) {
						uni.hideLoading();
						ths.setData({
							collectionCount: res
						});
					}
				};
				http.request(params);
			},

			/**
			 * 我的收藏跳转
			 */
			myCollectionHandle: function() {
				var url = '/pages/prod-classify/prod-classify?sts=5';
				var id = 0;
				var title = "我的收藏商品";

				if (id) {
					url += "&tagid=" + id + "&title=" + title;
				}

				uni.navigateTo({
					url: url
				});
			},

			/**
			 * 去登陆
			 */
			toLogin: function() {
				uni.navigateTo({
					url: "../accountLogin/accountLogin"
				})
			},

			/**
			 * 退出登录
			 */
			logout: function() {
				util.removeTabBadge()
				uni.removeStorageSync('loginResult');
				uni.removeStorageSync('token');
				
				// this.$Router.pushTab('/pages/index/index')
				uni.showToast({
					title: "退出成功",
					icon: "none"
				})
				
				this.setData({
					orderAmount: ''
				});
				setTimeout(() => {
					uni.switchTab({
						url: "/pages/index/index"
					})
				}, 1000)
			},
			toUserInfoPage: function(e) {
				uni.navigateTo({
					url: "../userInfo/userInfo"
				})
			}
		}
	};
</script>
<style>
	@import "./user.css";
</style>
