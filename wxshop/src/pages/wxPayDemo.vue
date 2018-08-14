<template>

  <transition name="page">
    <section class="ck-doc orderDetails">
          <div class="od-ft" @click="wxConfig();">立即支付</div>

          <div class="od-ft" @click="wxGetDate('code');">获取code</div>

          <div class="od-ft" @click="wxGetDate('access_token');">获取access_token</div>

          <div class="od-ft" @click="wxGetDate('jsapi_ticket');">获取jsapi_ticket</div>
    </section>
  </transition>

</template>
<style type="text/css">
      .od-ft{
        /*width: 100px;*/
        height: 50px;
        padding: 0 10px;
        background-color: red;
        color: #fff;
        text-align: center;
        line-height: 50px;
        font-size: 24px;
        margin-bottom: 20px;
      }
  </style>
<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,getStore,setStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import wx from 'weixin-js-sdk';
export default {
  data() {
    return {
      title: '订单详情',
      optsOrderBandReceiver: {
        couponIds:[],//优惠券集合
        orderId:"7f0000015f2a1075815f2d4cdf8b00a5",
        receiverId:"",
        reducedId:''//满减送id
      },
      apiOrderBandReceiverUrl: window.baseUrl+window.appName+'/shopapi/func/product/orderBandReceiver?orgCode='+window.orgCode,
      payType: "1",
      payMoney: "",
      orderNumber: "",
      orderListData: {orderNumber:'DD2017082916185024'},
      wxOpt:{}
    }
  },
  watch: {
      '$route.query.source': 'markInit'  //监听当前路由变化（辅助初始化）
    },
  beforeMount() {
    //挂载完毕初始化页面数据
    this.markInit();
  },
  methods: {
    
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
       
      console.log(wx);
       
    },
     
    wxConfig(){
      //微信支付配置
      var that = this;
      // wx.config({
      //     debug: false, 
      //     appId: this.wxOpt.appId,
      //     timestamp: this.wxOpt.timeStamp, //生成签名的时间戳
      //     nonceStr: this.wxOpt.nonceStr, //生成签名的随机串
      //     signature: "MD5",
      //     jsApiList: ['chooseWXPay']
      // });
      console.log('wxPay');
      wx.config({
          debug: false, 
          appId: "wx122c017d1608c62a",
          timestamp: "1506020669", //生成签名的时间戳
          nonceStr: "1506020669522", //生成签名的随机串
          signature: "MD5",
          jsApiList: ['chooseWXPay']
      });
      wx.ready(function(){
        console.log('wxConfigSucces');
        // wx.checkJsApi({
        //     jsApiList: ['chooseWXPay'], // 需要检测的JS接口列表，所有JS接口列表见附录2,
        //     success: function(res) {
        //         // 以键值对的形式返回，可用的api值true，不可用为false
        //         // 如：{"checkResult":{"chooseImage":true},"errMsg":"checkJsApi:ok"}
        //         // console.log(res);
        //     }
        // });
        that.onBridgeReady();
      });
      wx.error(function(res){
        console.log('wxConfigError');
      });
    },
    onBridgeReady() {
      //调用微信支付（弹窗支付）
       var that = this;
        wx.chooseWXPay({
          // timestamp: this.wxOpt.timeStamp,
          // nonceStr: this.wxOpt.nonceStr, // 支付签名随机串，不长于 32 位
          // package: this.wxOpt.package, // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
          // signType: "MD5", // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
          // paySign: this.wxOpt.paySign, // 支付签名
          timestamp: '1506020669',
          nonceStr: "1506020669522", // 支付签名随机串，不长于 32 位
          package: "prepay_id=wx20170922030429a38a8986c00595055877", // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
          signType: "MD5", // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
          paySign: 'A92FD1E112AE9752F045BB957011B712', // 支付签名
          success: function (res) {
            console.log("=>1"+res);
              // 支付成功后的回调函数
          },
          fail: function (res) {
            console.log("=>2", res);
          },
          complete: function (res) {
            console.log({"=>3":res});
          }
        });
    },
    getWxPayOpts() {
      //请求服务器获取微信支付需要相关参数
        let that = this;
        that.orderNumber = that.orderListData.orderNumber;
        let aipWxpayUrl = window.shopUrl+window.appName+'/wxpay/pay?orderNo='+that.orderNumber+'&money='+that.payMoney+'&openId='+window.openId+'&shopsid='+window.shopsid+'&orgCode='+window.orgCode +'&payMType='+ that.payType;
        that.$dialog.loading.open('请求数据中...');
        that.$http.get(aipWxpayUrl)
        .then((response) => {
          that.isApply = false;
          const data = response.data;
          that.$dialog.loading.close();
          if(data.code == "S_OK") {
            console.log(data.var);
            that.wxOpt = data.var;
            that.wxConfig();
             // that.onBridgeReady(data.var.appId, data.var.timeStamp, data.var.nonceStr, data.var.package, data.var.paySign);
          }else {
            that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
          }
        });
    },
    orderBandReceiver() {
      var that = this;
      console.log({'type':'orderDetail','opt':that.optsOrderBandReceiver});
      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.apiOrderBandReceiverUrl+'&shopsid='+window.shopsid,that.optsOrderBandReceiver)
        .then((response) => {
          const data = response.data;
          that.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.payMoney = data['var'].actualPrice;
            that.payType = data['var'].buyType;
            this.getWxPayOpts();
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
          that.isApply = false;
        },(response) => {
          that.$dialog.loading.close();
          that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
        });
    }
  } 
}
</script>
