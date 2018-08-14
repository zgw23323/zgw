<template>
  <section class="ck-doc action_bg1">
    <article class="order_content">
      <!-- 代金券头部 -->
      <div class="item_top" v-if="type==2">
        <div class="coupon i_flex">
          <div class="left">
            <div>￥<span class="font1">{{orderData.couponAmount}}</span><span class="font2 t">满{{orderData.leastConsumpMoney}}使用</span></div>
            <div class="">{{orderData.couponName}}</div>
            <div class="font2">有效期至{{orderData.endTime}}</div>
          </div>
          <div class="center">&nbsp;</div>
          <div class="right">
            <router-link :to="{path:'myCardCoupons'}">
              <div class="i_jian">&nbsp;</div>
              <div>查</br>看</div>
            </router-link>
          </div>
        </div>
        <div class="yuan yuan_1">&nbsp;</div>
        <div class="yuan yuan_2">&nbsp;</div>
      </div>
      <!-- 实物头部 -->
      <div class="item_top" v-if="type==1">
        <div class="product">
          <h4>出示您的验证码/二维码给工作人员，即可领取商品噢，
          还可以获得微信现金红包噢</h4>
        </div>
        <div class="yuan yuan_3">&nbsp;</div>
        <div class="yuan yuan_4">&nbsp;</div>
      </div>
      <div class="item_bottom">
        <!-- 代金券提示 -->
        <div v-if="type==2">
          <h3>您的代金券可以在会员中心，如下界面中，</br>“卡券”中查看噢</h3>
          <div class="car_img">&nbsp;</div>
        </div>
        <!-- 实物验证码 -->
        <div class="pro_code" v-if="type==1">
          <div v-if="orderData.redeemCode" :class="{'i_flex_center code_input':orderData.redeemStatus==1,'i_flex_center code_input num_has':orderData.redeemStatus==2}">
            <div class="p">{{orderData.redeemCode.substring(0,1)}}</div>
            <div class="p">{{orderData.redeemCode.substring(1,2)}}</div>
            <div class="p">{{orderData.redeemCode.substring(2,3)}}</div>
            <div class="p">{{orderData.redeemCode.substring(3)}}</div>
          </div>
          <div :class="{'code_img code_has':orderData.redeemStatus==2,'code_img':orderData.redeemStatus==1}" :style="{backgroundImage: 'url(' + imgBaseUrl+orderData.qrCodeUrl + ')'}">&nbsp;</div>
          <div class="dui_logo" v-if="orderData.redeemStatus==2"><i class="h_logo">&nbsp;</i></div>
        </div>
        <div class="i_flex_center a_suggest">
          <span class="line">&nbsp;</span><span>小主，给个建议吧</span><span class="line">&nbsp;</span>
        </div>
        <div class="xin i_flex_center">
          <span :class="{'cur':item.isBtn,'no':!item.isBtn}" v-if="applyOpt.starCount!= 0" v-for="item,index in xinList">&nbsp;</span>
          <span :class="{'cur':item.isBtn,'no':!item.isBtn}" v-if="applyOpt.starCount == 0" v-for="item,index in xinList" @click="xinBtn(index)">&nbsp;</span>
        </div>
        <div class="s_text">
          <textarea type='text' v-if="applyOpt.commentDesc" disabled readonly="readonly" v-model="applyOpt.commentDesc" placeholder="小主，对本次活动，您有什么想说的呢？"></textarea>
          <textarea type='text' v-else v-model="commentDesc" placeholder="小主，对本次活动，您有什么想说的呢？"></textarea>
        </div>
        <div class="a_submit" @click="enterSonTask" v-if="!applyOpt.commentDesc">&nbsp;</div>
      </div>
    </article>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {wxConfig} from '../../utils/wxSDK';
  export default {
    data() {
      return {
        title: '抢购成功',
        isShow:false,
        imgBaseUrl: window.imgBaseUrl,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/recharge/wxListRecharge',
        apiOrderInfoUrl: window.baseUrl+window.appName+"/shopapi/func/springwarm/wxYiyuanOrderInfo?sid="+window.shopsid,
        apiYiyuanCommentUrl: window.baseUrl+window.appName+'/shopapi/func/springwarm/wxYiyuanComment?sid='+window.shopsid,
        apiStatusUrl: window.baseUrl+window.appName+"/shopapi/func/springwarm/wxYiyuanRedeemStatus?shopsid="+window.shopsid,
        xinList:[
          {id:0,isBtn:false},{id:0,isBtn:false},{id:0,isBtn:false},{id:0,isBtn:false},{id:0,isBtn:false}
        ],
        type:'',
        applyOpt:{
          detailId:'1',
          starCount:0,
          commentDesc:''
        },
        commentDesc:'',
        orderData:{}
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchOrderInfo();
      this.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_ACTIONORDERINFO});
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
        //获取shopsid缓存数据
        setShopsId();
      },
      //获取二维码扫码状态
      getCodeStatu(){
        let that = this;
        that.$http.post(that.apiStatusUrl,{redeemCode:this.orderData.redeemCode})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(data.var){
                that.orderData.redeemStatus = 2;
                clearInterval(this.timer);
              }
            } 
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      run() {
        this.timer = setInterval(() => {
          this.getCodeStatu()
        }, 2000);
      },
      //星星点击
      xinBtn(n){
        this.xinList[n].isBtn = !this.xinList[n].isBtn;
        if(this.xinList[n].isBtn){
          for(var i=0;i<n;i++){
            this.xinList[i].isBtn = true;
          }
        }else{
          for(var i=n;i<this.xinList.length;i++){
            this.xinList[i].isBtn = false;
          }
        }
      },
      enterSonTask(){
        var that = this;
        var len = this.xinList.filter(function(data){
          return data.isBtn;
        });
        this.applyOpt.commentDesc = this.commentDesc;
        this.applyOpt.starCount = len.length;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiYiyuanCommentUrl,that.applyOpt)
          .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              this.$dialog.toast({mes: '留言成功', icon: 'success', timeout: 2000});
            }else {
              // this.$dialog.notify({mes: "网络异常，请稍后再来解绑~", icon: 'error', timeout: 1000});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      //获取订单详情
      fetchOrderInfo(){
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiOrderInfoUrl,{orderId:this.$route.query.id})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.type = data.var.structureType;
              that.orderData = data.var.structureType==1? data.var.shiwuOrderInfo:data.var.couponInfo;
              that.applyOpt.detailId = data.var.detailId;
              that.applyOpt.commentDesc = data.var.commentDesc;
              that.applyOpt.starCount = data.var.starCount;
              if(data.var.starCount){
                for(var i=0;i<data.var.starCount;i++){
                  that.xinList[i].isBtn = true;
                }
              }
              if(that.orderData.redeemStatus && data.var.structureType==1){
                this.run();
              }
              setTimeout(()=>{
                that.$dialog.loading.close();
              },300);
            }else {
              // this.$dialog.notify({mes: "网络异常，请稍后再来解绑~", icon: 'error', timeout: 1000});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    },
    destroyed() {
      clearInterval(this.timer);
    }
  }
                
</script>