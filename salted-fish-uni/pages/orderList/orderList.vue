<template>
<!--pages/orderList/orderList.wxml-->

<view class="container">
  <!-- 头部菜单 -->
  <view class="order-tit">
    <text @tap="onStsTap" data-sts="0" :class="sts==0?'on':''">全部</text>
    <text @tap="onStsTap" data-sts="1" :class="sts==1?'on':''">待支付</text>
    <text @tap="onStsTap" data-sts="2" :class="sts==2?'on':''">待发货</text>
    <text @tap="onStsTap" data-sts="3" :class="sts==3?'on':''">待收货</text>
	<text @tap="onStsTap" data-sts="5" :class="sts==5?'on':''">已取消</text>
    <text @tap="onStsTap" data-sts="4" :class="sts==4?'on':''">已完成</text>
  </view>
  <!-- end 头部菜单 -->
  <view class="main">
    <view class="empty" v-if="orderList.length==0">
      还没有任何相关订单
    </view>
    <!-- 订单列表 -->
    <block v-for="(item, index) in orderList" :key="index">
      <view class="prod-item">
        <view class="order-num">
          <text>{{item.sellerName}}</text>
          <view class="order-state">
            <text :class="'order-sts  ' + (item.status==1?'red':'') + '  ' + ((item.status==5||item.status==6)?'gray':'')">{{item.status==1?'待支付':(item.status==2?'待发货':(item.status==3?'待收货':(item.status==5?'已取消':'已完成')))}}</text>

            <view class="clear-btn" v-if="item.status==5 || item.status==6">
              <image src="/static/images/icon/clear-his.png" class="clear-list-btn" @tap="delOrderList" :data-ordernum="item.orderId"></image>
            </view>
          </view>
        </view>

        <!-- 商品列表 -->
        <!-- 一个订单单个商品的显示 -->
        <block v-if="item.orderDetails.length==1">
          <block v-for="(prod, index2) in item.orderDetails" :key="index2">
            <view>
              <view class="item-cont" @tap="toOrderDetailPage" :data-ordernum="item.orderId">
                <view class="prod-pic">
                  <image :src="serverUrl+prod.image"></image>
                </view>
                <view class="prod-info">
                  <view class="prodname">
                    {{prod.title}}
                  </view>
                  <view class="prod-info-cont">{{prod.num}}</view>
                  <view class="price-nums">
                    <text class="prodprice"><text class="symbol">￥</text>
                    <text class="big-num">{{wxs.parsePrice(prod.price)[0]}}</text>
                    <text class="small-num">.{{wxs.parsePrice(prod.price)[1]}}</text></text>
                    <text class="prodcount">x{{prod.num}}</text>
                  </view>
                </view>
              </view>
            </view>
          </block>
        </block>
        <!-- 一个订单多个商品时的显示 -->
        <block v-else>
          <view class="item-cont" @tap="toOrderDetailPage" :data-ordernum="item.orderId">
            <scroll-view scroll-x="true" scroll-left="0" scroll-with-animation="false" class="categories">
              <block v-for="(prod, index2) in item.orderDetails" :key="index2">
                <view class="prod-pic">
                  <image :src="serverUrl+prod.image"></image>
                </view>
              </block>
            </scroll-view>
          </view>
        </block>

        <view class="total-num">
          <text class="prodcount">共1件商品</text>
          <view class="prodprice">合计：
            <text class="symbol">￥</text>
            <text class="big-num">{{wxs.parsePrice(item.actualPay)[0]}}</text>
            <text class="small-num">.{{wxs.parsePrice(item.actualPay)[1]}}</text>
          </view>
        </view>
        <!-- end 商品列表 -->
        <view class="prod-foot">
          <view class="btn">
            <text v-if="item.status==1" class="button" @tap="cancelOrder" :data-ordernum="item.orderId" hover-class="none">取消订单</text>
            <text class="button warn" @tap :data-ordernum="item.orderId" hover-class="none">再次购买</text>
            <text v-if="item.status==1" class="button warn" @tap="normalPay" :data-ordernum="item.orderId" hover-class="none">付款</text>
            <text v-if="item.status==3 || item.status==4" class="button" @tap="toDeliveryPage" :data-ordernum="item.orderId" hover-class="none">查看物流</text>
            <text v-if="item.status==3" class="button warn" @tap="onConfirmReceive" :data-ordernum="item.orderId" hover-class="none">确认收货</text>
          </view>
        </view>
      </view>



    </block>


  </view>
</view>
<!-- end 订单列表 -->
</template>

<script module="wxs" lang="wxs" src="../../wxs/number.wxs"></script>

<script>
var http = require("../../utils/http.js");
var config = require("../../utils/config.js");

export default {
  data() {
    return {
      orderList: [],
      current: 1,
      pages: 0,
      sts: 0,
	  serverUrl: ''
    };
  },

  components: {},
  props: {},

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    if (options.sts) {
      this.setData({
        sts: options.sts,
		serverUrl: config.domain
      });
      this.loadOrderData(options.sts, 1);
    } else {
      this.loadOrderData(0, 1);
    }
  },

  /**
   * 生命周期函数--监听页面初次渲染完成
   */
  onReady: function () {},

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {},

  /**
   * 生命周期函数--监听页面隐藏
   */
  onHide: function () {},

  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {},

  /**
   * 页面相关事件处理函数--监听用户下拉动作
   */
  onPullDownRefresh: function () {},

  /**
   * 页面上拉触底事件的处理函数
   */
  onReachBottom: function () {
    if (this.current < this.pages) {
      this.loadOrderData(this.sts, this.current + 1);
    }
  },

  /**
   * 用户点击右上角分享
   */
  onShareAppMessage: function () {},
  methods: {
    /**
     * 加载订单数据
     */
    loadOrderData: function (sts, current) {
      var ths = this;
      uni.showLoading(); //加载订单列表

      var params = {
        url: "/order/list",
        method: "GET",
		needToken: true,
        data: {
          page: current,
          rows: 10,
          status: sts
        },
        callBack: function (res) {
          //console.log(res);
          var list = [];
          if (res.data.pageNum == 1) {
            list = res.data.list;
          } else {
            list = ths.orderList;
            Array.prototype.push.apply(list, res.data.list);
          }
          ths.setData({
            orderList: list,
            pages: res.data.pages,
            current: res.data.pageNum
          });
          uni.hideLoading();
        }
      };
      http.request(params);
    },

    /**
     * 状态点击事件
     */
    onStsTap: function (e) {
      var sts = e.currentTarget.dataset.sts;
      this.setData({
        sts: sts
      });
      this.loadOrderData(sts, 1);
    },

    /**
     * 查看物流
     */
    toDeliveryPage: function (e) {
      uni.navigateTo({
        url: '/pages/express-delivery/express-delivery?orderNum=' + e.currentTarget.dataset.ordernum
      });
    },

    /**
     * 取消订单
     */
    cancelOrder: function (e) {
      var ordernum = e.currentTarget.dataset.ordernum;
      var ths = this;
      uni.showModal({
        title: '',
        content: '要取消此订单？',
        confirmColor: "#3e62ad",
        cancelColor: "#3e62ad",
        cancelText: '否',
        confirmText: '是',
        success(res) {
          if (res.confirm) {
            uni.showLoading({
              mask: true
            });
            var params = {
              url: "/order/"+ordernum+"/5",
              method: "PUT",
			  needToken: true,
              data: {},
              callBack: function (res) {
                //console.log(res);
                ths.loadOrderData(ths.sts, 1);
                uni.hideLoading();
              }
            };
            http.request(params);
          } else if (res.cancel) {//console.log('用户点击取消')
          }
        }

      });
    },
		//模拟支付，直接提交成功
		normalPay: function(e){
			uni.showLoading({
			  mask: true
			});
			var params = {
			  url: "/order/"+e.currentTarget.dataset.ordernum+"/2",
			  method: "PUT",
			  needToken: true,
			  callBack: res => {
					console.log("res",res)
					uni.hideLoading();
					if(res){
						uni.showToast({
							title: "模拟支付成功",
							icon:"none"
						})
						setTimeout(() => {
							uni.navigateTo({
							  url: '/pages/pay-result/pay-result?sts=1&orderNumbers=' + e.currentTarget.dataset.ordernum
							});
						},1200)
					}else{
						uni.showToast({
							title: "支付失败！",
							icon:"none"
						})
					}
			  }
			};
			http.request(params);
		},

    /**
     * 查看订单详情
     */
    toOrderDetailPage: function (e) {
      uni.navigateTo({
        url: '/pages/order-detail/order-detail?orderNum=' + e.currentTarget.dataset.ordernum
      });
    },

    /**
     * 确认收货
     */
    onConfirmReceive: function (e) {
      var ths = this;
      uni.showModal({
        title: '',
        content: '我已收到货？',
        confirmColor: "#eb2444",

        success(res) {
          if (res.confirm) {
            uni.showLoading({
              mask: true
            });
            var params = {
              url: "/order/"+e.currentTarget.dataset.ordernum+"/4",
              method: "PUT",
			  needToken: true,
              data: {},
              callBack: function (res) {
                //console.log(res);
                ths.loadOrderData(ths.sts, 1);
                uni.hideLoading();
              }
            };
            http.request(params);
          } else if (res.cancel) {//console.log('用户点击取消')
          }
        }

      });
    },
    //删除已完成||已取消的订单
    delOrderList: function (e) {
      var ths = this;
      uni.showModal({
        title: '',
        content: '确定要删除此订单吗？',
        confirmColor: "#eb2444",

        success(res) {
          if (res.confirm) {
            var ordernum = e.currentTarget.dataset.ordernum;
            uni.showLoading();
            var params = {
              url: "/order/" + ordernum,
              method: "DELETE",
			  needToken: true,
              callBack: function (res) {
                ths.loadOrderData(ths.sts, 1);
                uni.hideLoading();
              }
            };
            http.request(params);
          } else if (res.cancel) {
            console.log('用户点击取消');
          }
        }

      });
    }
  }
};
</script>
<style>
@import "./orderList.css";
</style>