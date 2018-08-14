<template>

	<transition name="page">
		<section class="ck-doc orderDetails">

		<div v-if="!tips1">
        <div class="od-goods-wrap" v-if="odshow">
          <div class="od-goods-address" v-if="orderListData.isExpress">
            <div class='od-goods-address-top'>
              <div class="address-mess">
                <div class="add_logo">&nbsp;</div>
                <div v-if="hasAddress">
                  <p class="p1"> <span class='add-name'>{{express.consignee}}</span> {{express.phone}}</p>
                  <p class="p2">{{express.fullAddress}}</p>
                  <!-- <p class="p3" @click="addressChage">更改地址</p> -->
                </div>
                <div v-if="!hasAddress">
                  <p class="p3" @click="addressChage">新增地址</p>
                </div>
                <div v-if="hasAddress" class="address_right_btn" @click="addressChage" style="margin-top: .19rem"><slot name="right"></slot></div>
                 <div v-else class="address_right_btn" @click="addressChage" style="margin-top: 0"><slot name="right"></slot></div>
              </div>
            </div>
            <div class="address_line">&nbsp;</div>
          </div>
  		    <ul class="od-goods">
  		      <li v-for="details in orderListData.detailList">
  		      	<div class="ord-mess">
  		      		<div class="od-goods-img imgLazy" v-lazy:background-image="imgBaseUrl+details.prodPicUrl" @click="detailsGo(details.prodId)"></div>
  			        <div class="od-goods-mes">
  			          <p class="od-goods-title" v-text="details.prodName" v-if="details.serveCode ==''"></p>
                  <p class="od-goods-title" v-else-if="details.serveCode !=''&& details.serveType==0">{{details.prodName}} ({{details.serveCount}}次)</p>
                  <p class="od-goods-title" v-else>{{details.prodName}} (无限次)</p>
                  <p class="od-goods-format">{{details.prodGuigeDesc}}</p>
  			          <p class="od-goods-memberPrice" v-if='orderListData.fromSource==3'>￥{{details.discountPrice}}</p>
                  <p class="od-goods-memberPrice" v-else-if='orderListData.fromSource==2'>￥{{details.spikePrice}}</p>
                  <p class="od-goods-memberPrice" v-else>￥{{details.memberPrice}}</p>
  			        </div>
  		        </div>
  		        <span class="od-goods-count">×{{details.prodCount}}</span>
  		      </li>
  		    </ul>
        </div>

        <div class="od-u-points" v-if="orderListData.fromSource!=3 && orderListData.fromSource!=2 && serveCode==''">
          使用积分：<span>{{orderListData.totalPoints}}分</span><span class="s1">({{orderListData.userPoints}}分)</span>
          <span :class="{'s2':orderListData.userPoints>=orderListData.totalPoints}" v-if="totalCurPrice>0">抵扣{{totalCurPrice}}元</span>
        </div>
        <div class="od_coupon_mess" @click='tipOpen(3)' v-if="(couponList.length> 0 && !orderListData.isContinueCoupon) || orderListData.isContinueCoupon">
          <span>优惠券：</span>
          <div v-if="!orderListData.isContinueCoupon" style="display: flex;">
            <span class="od_c_name">{{couponSelectName}}</span>
            <span class='right'>&nbsp;</span>
          </div>
          <div v-if="orderListData.isContinueCoupon" style="display: flex;">
            <span class="od_c_name">- {{orderListData.continueCouponList[0].couponAmount}}元</span>
          </div>
        </div>
        <div class="od-u-points" v-if="orderListData.reducedInfo !=null && orderListData.reducedInfo!='undefined'">
            满减送活动：
            <span class="od_cut" v-if='orderListData.reducedInfo.setType==0'>满{{orderListData.reducedInfo.threshold}}元减{{orderListData.reducedInfo.reducedMoney}}</span>
            <span class="od_cut" v-if='orderListData.reducedInfo.setType==1'>满{{orderListData.reducedInfo.threshold}}元送{{orderListData.reducedInfo.methods[0].couponAmount}}优惠券</span>
        </div>
        <!-- 页面调整-->
        <div class='od-all-price' v-if="orderListData.userPoints>=orderListData.totalPoints">
          <div>
              <span class='od-pro-count'>共计{{orderListData.detailList.length}}件商品</span><span class="c1">小计：</span>
              <span class="c2">￥{{orderTotalPrice}}</span>
          </div>
          <p v-if="totalCouponPrice>0">(共减免{{totalCouponPrice}}元现金)</p>
        </div>
        <div class='od-all-price' v-if="orderListData.userPoints<orderListData.totalPoints">
          <div>
              <span class='od-pro-count'>共计{{orderListData.detailList.length}}件商品</span><span class="c1">小计：</span>
              <span class="c2">￥{{orderTotalPrice}}</span>
          </div>
          <p v-if="totalCouponPrice>0">(共减免{{totalCouponPrice}}元现金)</p>
          <p v-else-if="serveCode ==''">(无可用积分)</p>
        </div>
		    <div class="od-ft-price-bar" v-if="odshow">
		      <div class="inner">
		        <div class="od-ft-price-btn od-ft-price-btn2-1">
              <span class="p1">共计:</span>
              <span class="p2">￥{{orderTotalPrice}}</span>
            </div>
		        <div :class="{'od-ft-price-btn od-ft-price-btn2-2':!isApply,'od-ft-price-btn od-ft-price-btn2-2 od-ft-price-hasBtn':isApply}" @click="submit">立即支付</div>
		      </div>
		    </div>
	     </div>

	    <div class="tipsPage" v-if="tips1">
	      <div class="tipsBox">
	        <div class="tipStatus" data-type="order"></div>
	        <p>亲，您没有相关订单详情噢~~</p>
	        <router-link to="/goods" class="btns">去逛逛</router-link>
	      </div>
	    </div>
      <!-- 选择 -->
      <yd-bottom-tip v-if="tips3" :isShow='true' :tipsType='"tips3"'>
        <ul class="od-full-song">
          <li class="item1" v-if="couponList.length>0" v-for="coupon in couponList" @click='couponCheck(coupon.couponId,coupon.couponAmount)'><em :class="{'icon_checked c_no':coupon.couponId !=couponSelectId,'icon_checked c_ing':coupon.couponId ==couponSelectId}">&nbsp;</em><span>省{{coupon.couponAmount}}元，{{coupon.couponName}}</span></li>
          <li v-else><span>暂无可用优惠券</span></li>
        </ul>
        <div class="ck-fixed-btn" @click='tipClose(3)'>确认选择</div>
      </yd-bottom-tip>
	  </section>
	</transition>

</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,getStore,setStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
export default {
	data() {
		return {
			title: '订单详情',
      hasAddress: false,
			tips1: false,
      tips3: false,
			toast2: false,
			odshow: false,
      isCheck:false,
      couponSelectId:'',
      couponSelectName:'',
      couponSelectPrice:0,
			imgBaseUrl: window.imgBaseUrl,
			apiPayOrOrderDetailUrl: window.baseUrl+window.appName+'/shopapi/func/product/payOrOrderDetail?orgCode='+window.orgCode,
      optsPayOrOrderDetail: {
         orderId: '',
         dealType: '',
         payType: '',
         payMoney: '',
         payTime: '',
         wxPayCode: '',
         serialNumber: ''
      },
     	orderdata: {},
      isApply:false,//提交的时候会根据这个属性来判断是否已经提交过,已经提交过就退出，避免重复提交
     	detailsData: [],
     	apiOrderBandReceiverUrl: window.baseUrl+window.appName+'/shopapi/func/product/orderBandReceiver?orgCode='+window.orgCode,
     	optsOrderBandReceiver: {
        couponIds:[],//优惠券集合
        reducedId:''//满减送id
      },
     	payList: [],
     	payType: "1",
     	payMoney: "",
     	orderNumber: "",
      apiOrderListUrl: window.baseUrl+window.appName+'/shopapi/func/product/confirmOrderInfo?orgCode='+window.orgCode,
      optsOrderList: {
        orderId: 0
      },
       //获取订单优惠券列表
      apiProdCouponList:window.baseUrl+window.appName+'/shopapi/func/coupon/userOrderCouponList?orgCode='+ window.orgCode +'&shopsid='+ window.shopsid,
      totalCurPrice:0,//积分抵扣减免金额
      totalCouponPrice:0,//共减免金额
      orderListData: {},
      orderTotalPrice:0,//应付总额
      couponList:[],//优惠券列表
      serveCode:'',
      express: {
        consignee: '',
        phone: '',
        fullAddress: ''
      }
		}
	},
	watch: {
      '$route.query.source': 'markInit'  //监听当前路由变化（辅助初始化）
    },
	beforeMount() {
    //挂载完毕初始化页面数据
		this.markInit();
    this.fetchOrderList(this.log);
	}, 
	methods: {
    detailsGo(id) {
      //跳去详情页面
      if(this.serveCode==''){
        this.$router.push({
          path: '/details',
          query: {
            pid: id
          }
        });
      }else{
        this.$router.push({
          path: '/serviceDetail',
          query: {
            'pid': id,
            'serveCode':this.serveCode,
            'psource':1
          }
        });
      }
    },
    log(mod) {
      var sItem = {};
          sItem.orderid = this.orderListData.orderId, //商品订单id
          sItem.orderno = this.orderListData.orderNumber, //商品订单号
          sItem.totalfee = this.orderListData.userPoints>=this.orderListData.totalPoints?this.orderListData.totalpointPrice:this.orderListData.totalMemberPrice;//订单总金额
      mod = mod || LOGCODE.HOME.PAGE_ORDERDETAILS;
      this.$store.dispatch('log', {account:mod, OrderItem: sItem});
    },
    addressChage() {
      //跳去地址
      this.$router.push({
        path: '/address',
        query: {
          mark: 'change'
        }
      });
    },
	  markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
      this.serveCode = this.$route.query.serveCode?this.$route.query.serveCode:'';
      setTimeout(()=>{
	      this.odshow = true;
	    },600);
    },
  	toastClose(toast) {
      //关闭提示框
    	if(toast == 1) {
      		this.toast1 = false;
    	}else if(toast == 2) {
      		this.toast2 = false;
    	}
    },
    tipOpen(tips){
      this['tips'+tips] = true;
    },
    tipClose(tips){
      this['tips'+tips] = false;
    },
    selectPayType(type,isAllow,payMoney) {
      //选择支付类型
  		if(isAllow == 1) {
        this.payMoney = payMoney;
  			this.payType = type;
  		}/*else {
  			this.payType = 1;
  		}*/
	  },
    saveCart() {
      let _saveCart = getStore('_saveCart',true);
      let _newArr = [];
      if(typeof(_saveCart) !='boolean' &&_saveCart.length){
        for(var i = 0; i < this.orderListData.detailList.length; i++) {
          for(var j = 0; j < _saveCart.length; j++) {
            if(_saveCart[j].prodId != this.orderListData.detailList[i].prodId) {
              _newArr.push(_saveCart[j]);
            }
          }
        }
        setStore("_saveCart",_newArr);
        if(_newArr.length){
          this.$store.commit('UPDATE_GoodsCount', _newArr.length);
          setStore("_saveCartCount",_newArr.length);
        }
      }     
    },
    fetchOrderList(cb) {
      //获取订单详情列表
      let that = this;
      that.optsOrderList.orderId = that.$route.query.oid;
      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.apiOrderListUrl+'&shopsid='+window.shopsid,that.optsOrderList)
        .then((response) => {
          const data = response.data;
          that.tips1 = false;
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
          if(data.code == "S_OK") {
            if(data.var.orderStatus == 1) {
              that.orderListData = data.var;
              //总共减免金额
              if(that.orderListData.fromSource ==2){//来源：秒杀使用秒杀价
                that.orderTotalPrice = that.orderListData.totalSpikePrice;
              }else if(that.orderListData.fromSource ==3){//来源：限时折扣折扣价
                that.orderTotalPrice = that.orderListData.totalDiscountPrice;
              }else if(that.orderListData.userPoints>=that.orderListData.totalPoints && this.serveCode==''){//普通商品可用积分充足情况，使用积分价
                let _totalCurPrice = that.orderListData.totalMemberPrice-that.orderListData.totalpointPrice;
                that.orderTotalPrice = that.orderListData.totalpointPrice;
                if(parseInt(_totalCurPrice.toFixed(2).split('.')[1])>0){
                  that.totalCurPrice = _totalCurPrice.toFixed(2);
                }else{
                  that.totalCurPrice = _totalCurPrice;
                }
                that.totalCouponPrice = that.totalCurPrice;
              }else{//普通商品无可用积分情况使用会员价
                that.orderTotalPrice = that.orderListData.totalMemberPrice;
              }
              //满减送抵扣信息
              if(that.orderListData.reducedInfo !=null){
                that.totalCouponPrice = that.totalCouponPrice+that.orderListData.reducedInfo.reducedMoney;
                that.orderTotalPrice = that.orderTotalPrice - that.orderListData.reducedInfo.reducedMoney;       
              }
              //优惠券抵扣信息
              if(that.orderListData.isContinueCoupon){
                for(var i=0;i<that.orderListData.continueCouponList.length;i++){
                  var item = that.orderListData.continueCouponList[i];
                  that.optsOrderBandReceiver.couponIds[i] = item.id;
                  that.totalCouponPrice = that.totalCouponPrice + item.couponAmount;
                  that.orderTotalPrice = that.orderTotalPrice - item.couponAmount;
                }
              }
            }else {
              that.tips1 = true;
            }
            if(that.orderListData.isExpress) {
              if(that.orderListData.expressList.length > 0) {
                that.hasAddress = true;
                let i = sessionStorage.getItem("_addressIndex");
                if(i == '' || i === null || i === undefined) {
                  i = 0 ;
                }
                that.express.rid = data.var.expressList[i].id;
                that.express.consignee = data.var.expressList[i].consignee;
                that.express.phone = data.var.expressList[i].phone;
                that.express.fullAddress = data.var.expressList[i].provinceId + data.var.expressList[i].cityId + data.var.expressList[i].areaId + data.var.expressList[i].address;
              }else {
                that.hasAddress = false;
              }
            }
            
            //获取订单优惠券列表
            if(!that.orderListData.isContinueCoupon){
              that.getCouponList();
            }
            that.saveCart();
          }else{
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN"){
              that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
          if (typeof cb == 'function') {
            cb();
          }
        },(response) => {
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
        });
    },
    orderBandReceiver() {
      //订单绑定
    	let that = this;
      if(that.$route.query.source == 1 || that.$route.query.source == 2) {
        that.optsOrderBandReceiver.orderId = that.orderListData.orderId;
        if(that.orderListData.isExpress) {
          if(that.orderListData.expressList.length > 0) {
            that.optsOrderBandReceiver.receiverId = that.express.rid;
          }else {
            that.$dialog.notify({mes: '请新增一个收获地址哦~', icon: 'error', timeout: 1200});
            return;
          }
        }
      }else {
        return;
      }
      //判断是有选择了优惠券信息
      if(that.couponSelectId !=''){
        that.optsOrderBandReceiver.couponIds[0] = that.couponSelectId;
      }
      
      //判断是否有满减送信息
      if(that.orderListData.reducedInfo != null){
        that.optsOrderBandReceiver.reducedId = that.orderListData.reducedInfo.id;
      }
      that.$dialog.loading.open('加载数据中...');
    	that.$http.post(that.apiOrderBandReceiverUrl+'&shopsid='+window.shopsid,that.optsOrderBandReceiver)
        .then((response) => {
          const data = response.data;
          that.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.payMoney = data['var'].actualPrice;
            that.payType = data['var'].buyType;
            that.pay();
          }else{
            that.isApply = false;
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN"){
              that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
          
        },(response) => {
          that.$dialog.loading.close();
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
        });
    },
    getWxPayOpts() {
      //请求服务器获取微信支付需要相关参数
    		let that = this;
    		that.orderNumber = that.orderListData.orderNumber;
        console.log({'orderNumber':that.orderNumber,'payMoney':that.payMoney});      
    		// let aipWxpayUrl = window.shopUrl+window.appName+'/wxpay/finalpay?orderNo='+that.orderNumber+'&money='+that.payMoney+'&openId='+window.openId+'&shopsid='+window.shopsid+'&orgCode='+window.orgCode+'&payMType=' + that.payType;
        let aipWxpayUrl = window.shopUrl+window.appName+'/wxpay/finalpay?orderNo='+that.orderNumber+'&money='+that.payMoney+'&openId='+window.openId+'&shopsid='+window.shopsid+'&orgCode='+window.orgCode +'&payMType='+ that.payType;

        if (window.wxdebug) {
          alert("获取的订单号是：" + that.orderNumber);
          alert("发起支付请求的路径是: " + aipWxpayUrl);
        }
        if(parseFloat(that.payMoney)==0){//支付0元时直接调后台处理订单接口
          that.payOrOrderDetail(1);
        }else if(window.isWXpay == true){
          that.$dialog.loading.open('请求数据中...');
          that.$http.get(aipWxpayUrl)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();
            if(data.code == "S_OK") {
               that.onBridgeReady(data.var.appId, data.var.timeStamp, data.var.nonceStr, data.var.package, data.var.paySign);
            }else {
              that.isApply = false;
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
            }
          });
        }else {
          /*that.$dialog.loading.open('请求数据中...');
          setTimeout(()=>{
            that.$dialog.loading.close();
            that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:ok";
            that.payOrOrderDetail(1);
          },1000);*/
          //dealType = 1;
          var formData = {
            func:function(){
              that.payOrOrderDetail(1);
            },
            cancel:function(){
              that.isApply = false;
              console.log("取消了支付......");
              that.$dialog.notify({mes: '您已取消支付', icon: 'error', timeout: 1000});

              that.log(LOGCODE.HOME.BTN_PAGE_ORDERDETAILS_PAY_CANCEL);

            },
            data:{"paytype":2, "paymoneytype": this.payType},// 2表示商品, this.payType: 原价、会员价、会员积分价
            forms:{"orgCode": window.orgCode, "orderNo": that.orderNumber, "payMoney": that.payMoney, "vipCard":"xx"}
          }
          that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:ok";
          this.$store.dispatch('simulationPay', formData);
        }
    },
    pay() {
      this.log(LOGCODE.HOME.BTN_PAGE_ORDERDETAILS_PAY);
      //判断是否提交过，避免重复提交
      //支付
    	this.getWxPayOpts();
    },
    payOrOrderDetail(dealType) {
      //支付提交
    	let that = this;
	    that.optsPayOrOrderDetail.dealType = dealType;
    	that.optsPayOrOrderDetail.orderId = that.orderListData.orderId;
      that.optsPayOrOrderDetail.payMoney =  that.payMoney;
      that.optsPayOrOrderDetail.payType =  that.payType;
      that.optsPayOrOrderDetail.serialNumber = that.orderListData.orderNumber;
      that.$dialog.loading.open('提交数据中...');
      console.log("payOrOrderDetail=", that.optsPayOrOrderDetail);
      that.$http.post(that.apiPayOrOrderDetailUrl+'&shopsid='+window.shopsid,that.optsPayOrOrderDetail)
        .then((response) => {
          const data = response.data;
          that.$dialog.loading.close();
          if(data.code == "S_OK") {
          	//跳去支付详情
          	that.$router.replace({
          		path: '/orderbill',
          		query: {
          			oid: that.optsPayOrOrderDetail.orderId
          		}
          	});
          }else{
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN"){
              that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          that.$dialog.loading.close();
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
        });
    },
    onBridgeReady(appId, timeStamp, nonceStr, packageValue, paySign ) {
      //调用微信支付（弹窗支付）
       var that = this;
        WeixinJSBridge.invoke(
         'getBrandWCPayRequest', {
             "appId":appId,     //公众号名称，由商户传入
             "timeStamp":timeStamp,         //时间戳，自1970年以来的秒数
             "nonceStr":nonceStr, //随机串
             "package":packageValue,
             "signType":"MD5",         //微信签名方式：
             "paySign":paySign //微信签名
         },function(res) {
             if(res.err_msg == "get_brand_wcpay_request:ok" ) {
                that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:ok";
                that.optsPayOrOrderDetail.payTime = timeStamp;
                that.payOrOrderDetail(1);
             }else if(res.err_msg == "get_brand_wcpay_request:cancel" ) {
                 that.$router.replace({
                   path: '/obligations',
                   query: {
                     type: 2,
                     mark: 'end'
                   }
                 });
             }else if(res.err_msg == "get_brand_wcpay_request:fail" ) {
                 that.optsPayOrOrderDetail.wxPayCode = "get_brand_wcpay_request:fail";
                 that.optsPayOrOrderDetail.payTime = timeStamp;
                 that.payOrOrderDetail(1);
                //that.$router.replace('/payerror');
             }
             if(window.wxdebug == true) {
                that.$dialog.alert({mes: JSON.stringify(res)});
             }
            that.isApply = false;
             // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。
         }
       );
    },
    //获取优惠券列表
    getCouponList() {
      let that = this;
        console.log(12);
        that.$http.post(that.apiProdCouponList,{'orderId':that.orderListData.orderId})
        .then((response) => {
          const data = response.data;
          that.tips1 = false;
          if(data.code == "S_OK") {
            that.couponList = data['var'];
            if(that.couponList.length>0){
              that.couponSelectName = '请选择';
            }else{
              that.couponSelectName = '无可用';
            }
          }else{
            that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
          }
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    //选择优惠券
    couponCheck(id,pName){   
      if(this.couponSelectId == id){
        this.couponSelectId = '';
        this.couponSelectName ='请选择';
        this.orderTotalPrice = this.orderTotalPrice + pName;
        this.totalCouponPrice = this.totalCouponPrice - pName;
        this.couponSelectPrice = 0;
      }else{
        this.couponSelectId = id;
        this.couponSelectName ='- '+pName+'元';
        this.orderTotalPrice = this.orderTotalPrice + this.couponSelectPrice - pName;
        this.totalCouponPrice = this.totalCouponPrice - this.couponSelectPrice  + pName;
        this.couponSelectPrice = pName;
      }
    },
    submit() {
      this.log(LOGCODE.HOME.BTN_PAGE_ORDERDETAILS_PAY);
      //立即支付
    	let that = this;
      //判断是否提交过，避免重复提交
      if(that.isApply){
        return false;
      }else{
        that.isApply = true;
      }
    	that.orderBandReceiver();
    }
	},
  beforeDestroy() {
    //移除地址下标记录
    sessionStorage.removeItem("_addressIndex");
  }
}
</script>
