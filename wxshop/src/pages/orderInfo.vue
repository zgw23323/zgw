<template>
	<section class="ck-doc mybill">
		<div class="Orderinfo" v-if="!hide">
      <div class='top'>
        <div class="state-goods">
          <p class="title order_count" v-if="orderInfo.orderStatus == 1">待付款<span>剩余：<yd-countdown class="s1" :cursystem="detailsData.currentTime" :time="orderInfo.orderDeadTimeString" format=" {%h}:{%m}:{%s}"></yd-countdown></span></p>
          <p class="title" v-else-if="orderInfo.orderStatus == 2">已完成</p>
          <p class="title" v-else-if="orderInfo.orderStatus == 3">已取消</p>
          <p class="title" v-else-if="orderInfo.orderStatus == 4">发货中</p>
          <p class="title" v-else>已发货</p>
          <p class="WaybillNum" v-if="orderInfo.orderStatus==5">运单号：{{orderInfo.expressNumber}}</p>
        </div>
        <div class="state-express">
          <div class="top- flex-spacing" v-if="orderInfo.orderStatus != 1 && orderInfo.orderStatus != 3 && orderInfo.receiverId" @click="goToExpressStatu">
            <div>
              <p class="title" v-if="orderInfo.orderStatus == 4">您的订单发货中</p>
              <p class="title" v-else-if="orderInfo.orderStatus == 5">您的订单已揽件</p>
              <p class="title" v-else>您的订单已签收</p>
              <p class="time">{{orderInfo.payDate}}</p>
            </div>
            <div class="arrow-right" v-if="orderInfo.orderStatus != 4"></div>
          </div>
          <div class="bottom-" v-if="receiverInfo && receiverInfo.consignee">
            <div class="flex">
              <p class="name">{{receiverInfo.consignee}}</p>
              <p class="phone">{{receiverInfo.phone}}</p>
            </div>
            <p class="address">{{receiverInfo.provinceId+receiverInfo.cityId+receiverInfo.areaId+receiverInfo.streetId+receiverInfo.address}}</p>
          </div>
        </div>
      </div>
      <div class="bottom">
        <!-- <p class="order_shop">{{canshu8}}</p> -->
        <div class="goods flex" v-for="prodect in detailsData.prodDetailList" @click="detailsGo(prodect.prodId,prodect)">
          <img class="img" :src="imgBaseUrl+prodect.prodPicUrl">
          <div>
            <div class="top-">
              <p class="title">{{prodect.prodName}}</p>
              <p class="price" v-if='prodect.fromSource==3'>￥{{prodect.discountPrice}}</p>
              <p class="price" v-else-if='prodect.fromSource==2'>￥{{prodect.spikePrice}}</p>
              <p class="price" v-else>￥{{prodect.memberPrice}}</p>
              <!-- <p class="price">￥{{canshu10}}</p> -->
            </div>
            <div class="bottom-">
              <p class="title">{{prodect.prodGuigeDesc}}</p>
              <p class="number">{{prodect.prodCount}}</p>
            </div>
          </div>
        </div>
        <div class="info">
          <!-- <div class="flex-spacing">
            <p class="title">商品价格</p>
            <p class="val">￥{{canshu13}}</p>
          </div> -->
          <div class="flex-spacing">
            <p class="title">使用积分</p>
            <p class="val">{{orderInfo.usePoints}}</p>
          </div>
          <div class="flex-spacing">
            <p class="title">订单总价</p>
            <p class="val-">￥{{orderInfo.totalPrice}}</p>
          </div>
        </div>
        <div class="info">
          <div class="flex-spacing">
            <p class="title">订单编号</p>
            <p class="val">{{orderInfo.orderNumber}}</p>
          </div>
          <div class="flex-spacing">
            <p class="title">支付方式</p>
            <p class="val" v-if="orderInfo.payType==1">微信支付</p>
          </div>
          <div class="flex-spacing">
            <p class="title">下单时间</p>
            <p class="val-">{{orderInfo.createTimeString}}</p>
          </div>
        </div>
        <div class="info-" v-if="orderInfo.expressNumber">
          <div class="flex-spacing">
            <p class="title">配送方式</p>
            <p class="val">普通快递</p>
          </div>
          <!-- <div class="flex-spacing">
            <p class="title">快递公司</p>
            <p class="val">{{orderInfo.expressCompanyName}}</p>
          </div>
          <div class="flex-spacing">
            <p class="title">运单号</p>
            <p class="val">{{orderInfo.expressNumber}}</p>
          </div> -->
          <div class="flex-spacing">
            <p class="title">快递公司</p>
            <p class="val">{{expressCompanyName}}</p>
          </div>
          <div class="flex-spacing">
            <p class="title">运单号</p>
            <p class="val">{{orderInfo.expressNumber}}</p>
          </div>
        </div>
      </div>
      <div class="operation" v-if="orderInfo.orderStatus == 1">
        <span class="cancel_btn" @click="cancleOrder(orderInfo.id)">取消订单</span>
        <span class="button" @click="pay(orderInfo.id,orderInfo)">支付</span>
      </div>
      <div class="operation" v-else-if="orderInfo.orderStatus == 5 || orderInfo.orderStatus == 4">
        <!-- <span class="cancel_btn">取消订单</span> -->
        <span class="button" @click="submitReceiving(orderInfo.id)">确认收货</span>
      </div>
      <div class="operation" v-else>
        <span class="cancel_btn" @click="removeOrder(orderInfo.id)">移除</span>
        <span class="button" @click="detailsGo(detailsData.prodDetailList[0].prodId,detailsData.prodDetailList[0])">再次购买</span>
      </div>
    </div>
	</section>
</template>


<script type="text/babel">
import {setShopsId,setSessionStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
import ajaxModel from '../utils/ajaxModel';
import {emsCode} from '../utils/emsCode';
export default {
	data() {
		return {
			title: '订单详情',
      optsPayOrOrderDetail: {
         orderId: '',
         dealType: 2
      },
      hide: true,
      imgBaseUrl: window.imgBaseUrl, 
      orderInfo:{},
      receiverInfo: {},
      optsRemoveOrder: {
        orderIds: []
      },
      optsCancleOrder: {
        orderIds: []
      },
      detailsData: {},
      expressCompanyName: ''
		}
	},
  watch: {
    '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据
    console.log("Vip:",window.isVip);
    this.markInit();
    let that = this;
    that.optsPayOrOrderDetail.orderId = that.$route.query.oid;
    that.$dialog.loading.open('加载中...');
    ajaxModel.wxGetOrderInfo(this,(result)=>{
      that.hide = false;
      that.$dialog.loading.close();
      that.detailsData = result;
      that.orderInfo = result.orderInfo;
      that.receiverInfo = result.receiverInfo;
      for(var index in emsCode){
        if(emsCode[index].companyCode == that.orderInfo.expressCompanyName) {
          that.expressCompanyName = emsCode[index].companyName;
          break;
        }
      }
    },that.optsPayOrOrderDetail);
  },
  methods: {
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
      //获取门店信息
      this.shop = window.shop;

      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_ORDERINFO});
    },
    detailsGo(id,pOpt) {
      if(pOpt.fromSource != 5){
        //跳去详情
        this.$router.push({
          path: '/details',
          query: {
            pid: id
          }
        });
      }else{
        //跳去一元拍商品详情
        this.$router.replace({
          path: '/actionProductDetail',
          query: {
            pid: id     
          }
        });
      }
    },
    //点击去支付
    pay(id,pOpt) {
        if(pOpt.fromSource != 5){
          //跳去支付详情页面
          this.$router.push({
              path: '/orderdetails',
              query: {
                source: 2,
                oid: id
              }
          });
        }else{//一元抢拍去支付
          this.$router.push({
              path: '/actionProductDetail',
              query: {
                pid: pOpt.detailList[0].prodId,
                orderNumber: pOpt.orderNumber,
                orderId: id
              }
          });
        }
    },
    cancleOrder(id) {
      //取消订单
      let that = this
      that.optsCancleOrder.orderIds = [];
      that.optsCancleOrder.orderIds[0] = { "orderId": id };
      that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_ORDERINFO_CANCEL_APPLY});
      ajaxModel.cancleOrder(this,(result)=>{
        that.$dialog.notify({mes: '取消订单成功', icon: 'error', timeout: 1000});
        that.$router.back();
        that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_ORDERINFO_CANCEL_APPLY_OK});
      },that.optsCancleOrder)
    },
    removeOrder(id) {
      //移除订单
      this.$dialog.confirm({
        title: '确认移除订单？',
        mes: '移除后无法恢复订单信息',
        opts: () => {
          let that = this;
          that.optsRemoveOrder.orderIds = [];
          that.optsRemoveOrder.orderIds[0] = { "orderId": id };
          that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_ORDERINFO_REMOVE_APPLY});
          ajaxModel.removeOrder(this,(result)=>{
            that.$dialog.notify({mes: '移除订单成功', icon: 'error', timeout: 1000});
            that.$router.back();
            that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_ORDERINFO_REMOVE_APPLY_OK});
          },that.optsRemoveOrder)
        }
      });
    },
    //确认收货
    submitReceiving(id) {
      let that = this;
      that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_ORDERINFO_SUBMITRECEIVING_APPLY});
      ajaxModel.confirmExpressDelivery(this,(result)=>{
        that.$dialog.notify({mes: '确认收货成功', icon: 'error', timeout: 1500});
        setTimeout(()=>{
          that.$router.back();
        }, 1500);
        that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_ORDERINFO_SUBMITRECEIVING_APPLY_OK});
      },{orderId:id})
    },
    //去到订单追踪页面
    goToExpressStatu() {
      //跳去详情
      if(this.orderInfo.orderStatus != 4){
        setSessionStore('ExpressStatu_info',{expressCompanyName:this.orderInfo.expressCompanyName,expressNumber:this.orderInfo.expressNumber,expressTime:this.orderInfo.payDate});
        this.$router.push({
          path: '/emsStatus',
          query: {
            orderType: this.orderInfo.orderStatus
          }
        });
      }
    }
  }
}
</script>
