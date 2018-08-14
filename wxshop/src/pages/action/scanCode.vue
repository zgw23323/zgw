<template>
	<section class="scanCode">
    <div v-if="type=='code'">
      <div class="code_num i_flex_center">
        <span  id="input1" @click="tipCount(1)" v-model="num1">{{num1}}</span>
        <span  id="input2" @click="tipCount(2)" v-model="num2">{{num2}}</span>
        <span  id="input3" @click="tipCount(3)" v-model="num3">{{num3}}</span>
        <span  id="input4" @click="tipCount(4)" v-model="num3">{{num4}}</span>
      </div>
      <h3>输入车主出示的验证码，确实车主抢购的商品</h3>
      <div class="f_buttom i_flex_center">
        <div>
          <div class="code_icon i_flex_center" @click="fetchSaoCode"><i>&nbsp;</i></div>
          <div class="c_btn" @click="fetchSaoCode">扫一扫二维码</div>
        </div>
      </div>
    </div>

     <!--扫码弹窗内容-->
    <yd-popup v-model="tips2" position="center" width="5.8rem">
      <div class="p_tip">
        <div class="close_i" @click="closeTip(2)"><i>&nbsp;</i></div>
         <div class="i_info">
          <div class="i_img"><img :src="imgBaseUrl+productData.thumbnailUrl" /></div>
          <p>{{productData.prodTitle}}<span class="has" v-if="productData.isFirst==false">（已兑换）</span><span class="has" v-if="productData.isFirst ==true && productData.redeemStatus==2">（兑换成功）</span></p>
        </div>
      </div>
    </yd-popup>

    <yd-keyboard ref="child" v-model="tips3" :tipsType="'tips3'" v-show="tips3"></yd-keyboard>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {wxConfig} from '../../utils/wxSDK';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '二维码/验证码',
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/springwarm/wxYiyuanRedeem?sid='+window.shopsid+'&openId='+window.openId,
        apiCodeUrl:window.baseUrl+window.appName+"/wxpay/jsapi?targetURL=",//微信扫一扫获取签名
        num1:'',
        num2:'',
        num3:'',
        num4:'',
        type:'',
        tips2:false,
        tips3:false,
        imgBaseUrl: window.imgBaseUrl,
        productData:{
          prodTitle:'',
          thumbnailUrl:''
        },
        wxCodeOpt:{}
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      if(this.type=='sao'){
        this.setTmestamp();
      }else{
        this.tips3 = true;
      }
      this.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_SCANCODE});
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
     
      markInit() {
        //初始化页面标题
        document.title = this.title;
        this.type = this.$route.query.type;
        //获取shopsid缓存数据
        setShopsId();
      },
      setCode() {
        if(this.num1 && this.num2 && this.num3 && this.num4){
          this.fetchProduct();
          this.tips3 = false;
        }
      },
      getCode(n) {
        for(var i=1;i<5;i++){
          if(!this['num'+i]){
            this['num'+i] = n.toString();
            break;
          }
        }
        this.setCode();
      },
      closeTip(n){
        this['tips'+n]= false;
        if(n==2&&this.type=='sao'){
          this.setTmestamp();
        }
      },
      //验证扫一扫签名是否过期
      setTmestamp() {
        var curT = Math.floor(new Date().getTime() / 1000);
        if(this.wxCodeOpt.timestamp&&(parseInt(this.wxCodeOpt.timestamp)+3600>curT)){//未过期
          this.scanQRCodeViter();
        }else{//已过期
          this.fetchSaoCode();
        }
      },
      deleteCode() {
        for(var i=4;i>0;i--){
          if(this['num'+i]){
            this['num'+i] ='';
            break;
          }
        }
      },
      tipCount(n) {
        this.tips3  = true;
      },
      //获取微信扫一扫签名
      fetchSaoCode() {
        let that = this;
        var targetURL = location.href.split('#')[0];
        that.$http.post(that.apiCodeUrl+encodeURIComponent(targetURL))
          .then((response) => {
            const data = response.data;
            if (data.code === 'S_OK') {
              that.wxCodeOpt = data.var;
              that.scanQRCodeViter();
            }
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);

          },(response) => {
            setTimeout(()=>{
                that.$dialog.loading.close();
                that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
              },300);
          });
      },
      //扫一扫验证
      scanQRCodeViter() {
        let that = this;
        wx.config({
          debug: false, 
          appId: 'wx122c017d1608c62a',
          timestamp: that.wxCodeOpt.timestamp, //生成签名的时间戳
          nonceStr: that.wxCodeOpt.noncestr, //生成签名的随机串
          signature: that.wxCodeOpt.signature,
          jsApiList: ['scanQRCode']
        });
        wx.scanQRCode({
          desc: 'scanQRCode desc',
          needResult: 1, // 默认为0，扫描结果由微信处理，1则直接返回扫描结果，
          scanType: ["qrCode","barCode"], // 可以指定扫二维码还是一维码，默认二者都有
          success: function (res) {
            var strNum = res.resultStr.toString();
            that.num1 = strNum.substring(0,1);
            that.num2 = strNum.substring(1,2);
            that.num3 = strNum.substring(2,3);
            that.num4 = strNum.substring(3);
            that.fetchProduct();
          },
          error: function(res){
            if(res.errMsg.indexOf('function_not_exist') > 0){
              that.$dialog.notify({mes: '版本过低，请升级版本！', icon: 'error', timeout: 1200});
            }
          }
        });
      },
      //根据验证码查询物品
      fetchProduct() {
        let that = this;
        var opt = {redeemCode:this.num1+this.num2+this.num3+this.num4};
        that.$dialog.loading.open('加载中...');
        that.$http.post(that.apiUrl,opt)
          .then((response) => {
            this.$dialog.loading.close();
            const data = response.data;
            if (data.code === 'S_OK') {
              this.productData = data.var;
              that.tips2 = true;
            }else if(data.code=="REDEEM_CODE_ERROR"){
              that.$dialog.notify({mes: '请输入正确的验证码', icon: 'error', timeout: 2000});
            }
          },(response) => {
            setTimeout(()=>{
                that.$dialog.loading.close();
                that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
              },300);
          });
      }
    }
  }
</script>
