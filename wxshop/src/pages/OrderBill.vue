<template>

	<!--<transition name="page">-->
		<section id="mybill" class="ck-doc mybill" :class="hide ? 'mybillhide' : ''">
			<div id="mybillhd" class="mybill-hd">
        <div v-if="billData.orderInfo.transactStatus == 1">
				  <div class="mybill-icon" data-type="ordersuccess"></div>
				  <p class="mybill-icon-des">恭喜您，支付成功!</p>
        </div>
        <div v-if="billData.orderInfo.transactStatus == 2">
          <div class="mybill-icon" data-type="ordererror"></div>
          <p class="mybill-icon-des" data-type="ordererror">抱歉，交易失败!</p>
        </div>
        <div v-if="billData.orderInfo.transactStatus == 0">
          <div class="mybill-icon" data-type="ordererror"></div>
          <p class="mybill-icon-des" data-type="ordererror">您的订单已取消</p>
        </div>
			</div>
      <div id="mybillWrap">
        <div id="ndsp"></div>
        <div class="mybill-bd">
          <ul class="mybill-list" :class="billData.isExpress ? 'line' : ''">
            <li>订单编码：<span>{{billData.orderInfo.orderNumber}}</span></li>
            <li>商品名称：<span>{{billData.prodNames}}</span></li>
            <li v-if="billData.orderInfo.payType == 1 ? true : false">支付方式：<span>微信支付</span></li>
            <li>支付金额：<span>{{parseFloat(billData.orderInfo.actualPrice/100).toFixed(2)}}</span></li>
            <li v-if='billData.orderInfo.fromSource !=2 && billData.orderInfo.fromSource !=3'>使用积分：<span>{{billData.orderInfo.usePoints}}</span></li>
            <li v-if="billData.orderInfo.couponAmount>0">优惠券：<span>减{{billData.orderInfo.couponAmount}}元</span></li>
            <li v-if="billData.orderInfo.reducedAmount>0">满减送：<span>优惠{{billData.orderInfo.reducedAmount}}元</span></li>
            <!-- <li>剩余积分：<span>{{billData.orderInfo.usePoints}}</span></li> -->
            <li>下单时间：<span>{{billData.orderInfo.payStartDate}}</span></li>
          </ul>
          <div v-if="billData.isExpress">
            <p class="mybill-sh">收货信息</p>
            <ul class="mybill-list">
              <li>收件人：<span>{{billData.receiverInfo.consignee}}</span></li>
              <li>联系电话：<span>{{billData.receiverInfo.phone}}</span></li>
              <li>收货地址：<span>{{billData.receiverInfo.provinceId}}{{billData.receiverInfo.cityId}}{{billData.receiverInfo.areaId}}{{billData.receiverInfo.address}}</span></li>
            </ul>
          </div>
        </div>
      </div>
      <div id="mybillft" class="mybill-ft">
        <div class="inner">
          <p class="p1">{{shop.name}}</p>
          <p class="p2">{{shop.telphone}} {{shop.address}}</p>
        </div>
      </div>
		</section>
<!--	</transition>-->

</template>

<script type="text/babel">
import {setShopsId} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
export default {
	data() {
		return {
			title: '支付成功',
      hide: true,
			apiPayOrOrderDetailUrl: window.baseUrl+window.appName+'/shopapi/func/product/payOrOrderDetail?orgCode='+window.orgCode,
      optsPayOrOrderDetail: {
         orderId: '',
         dealType: 2
      },
      orderdata: {},
      shop: {
        name: '',
        telphone: '',
        address: ''
      },
      billData: {
        orderInfo: {
          orderNumber: ''
        }
      },
      detailsData: {}
		}
	},
  watch: {
    '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
    this.fetchBillList();
  },
  methods: {
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
      //获取门店信息
      this.shop = window.shop;

      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_ORDERBILL});
    },
    billHeightInit() {
      //初始bill高度
      document.getElementById("mybillWrap").style.minHeight = document.body.clientHeight - document.getElementById("mybillhd").offsetHeight - document.getElementById("mybillft").offsetHeight + 'px';
      this.hide = false;
    },
    fetchBillList() {
      //获取订单结果详情数据
      let that = this;
      that.$dialog.loading.open('加载数据中...');
      that.optsPayOrOrderDetail.orderId = that.$route.query.oid;
      that.$http.post(that.apiPayOrOrderDetailUrl+'&shopsid='+window.shopsid,that.optsPayOrOrderDetail)
        .then((response) => {
          const data = response.data;
          if(data.code == "S_OK") {
            that.billData = data.var;
          }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
            that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
              that.$router.push('/login');
            }});
          }else if(data.code == "NO_LOGIN") {
            that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
              that.$router.push('/login');
            }});
          }else {
            that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
          }
          setTimeout(()=>{
            that.$dialog.loading.close();
            that.billHeightInit();
          },300);
        },(response) => {
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    }
  }
}
</script>
