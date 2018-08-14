<template>
  <section class="ck-doc service" style="padding-bottom: 0;">
    <article class="header">
      <div class="head_vip">
        <div class="top">
          <div class="logo"><img :src="userInfo.userPic" v-if="userInfo.userPic !=''" /><img src="../../img/serviceImg/logo.png" v-else /></div>
          <!-- 会员身份 -->
          <div class="info" v-if="userInfo.userStatus==1 || isVip">
            <p class="p1">{{userInfo.userName}}</p>
            <p class="p2">会员卡号：{{userInfo.cardNumber}} <span @click="toastOpen(1)" class='go_buy'>(我的会员卡)</span></p>
          </div>
          <!-- 准会员身份 -->
          <div class="noKey" v-else-if= "userInfo.userStatus == 2">
            <p>{{userInfo.userName}}</p>
            <p class="p2">会员身份未激活 <span @click="toastOpen(2)" class='go_buy'>点击此处激活</span></p>
          </div>
         <!-- 非会员 -->
          <div class="noVip" v-else>
            <p>{{userInfo.userName}}</p>
            <div @click ="toastOpen(3)">领取会员卡</div>
          </div>
        </div>
        <div class="butom">
          <div>
              <p class="p4">积分：<span class="p3">{{userInfo.credits}}</span></p>
          </div>
          <div>
              <p class="p4">余额：<span class="p3">{{userInfo.balance}}</span></p>
          </div>
          <div class="car_item">
            <router-link :to="{path:'myCardCoupons'}" v-if="userInfo.couponCount">
              <p class="p4">卡券：<span class="p3">{{userInfo.couponCount}}</span><i class="c_dian">&nbsp;</i></p>
            </router-link>
            <p class="p4" v-else>卡券：<span class="p3">0</span></p>
          </div>
        </div>
      </div>
      <div class="head_nav">
        <a href="javascritt: void(0)" @click="toastOpen(2)">
          <div class="c_logo"></div>
          <p>会员入口</p>
        </a>
        <a href="javascritt: void(0)" @click="goTo('rechargeUrl')">
          <div class="v_logo"></div>
          <p>会员充值</p>
        </a>
        <a href="javascritt: void(0)" @click="goTo('wxHomeUrl')">
          <div class="s_logo"></div>
          <p>微信商城</p>
        </a>
      </div>
    </article>

    <article class="content" style="padding-bottom:0;">
      <div class="m_service">
        <p class="c_p">我的服务</p>
        <div class="service_list">
          <router-link :class="{'item':!serviceList.isHasDj,'itemDj':serviceList.isHasDj}" :index="index" key='index' :to="{path: 'serviceInfo',query:{'code':myServe.serveCode}}" v-for="(myServe,index) in serviceList.myServeDatas">
            <div :class="'i_com i_'+myServe.serveCode">&nbsp;</div>
            <p v-if="myServe.serveCode !='DJSERVICES'">{{myServe.serveName}}</p>
          </router-link>
         <!--  <router-link class='item' :index="index" key='index' :to="{path: 'serviceInfo',query:{'code':myServe.serveCode}}" v-for="(myServe,index) in serviceList.myServeDatas" v-if="myServe.serveCode !='DJSERVICES'">
            <div :class="'i_com i_'+myServe.serveCode">&nbsp;</div>
            <p>{{myServe.serveName}}</p>
          </router-link> -->
          <div :class="{'item':!serviceList.isHasDj,'itemDj':serviceList.isHasDj}" @click="goTo('violationQuery')">
              <div class="i_wei">&nbsp;</div>
              <p>违章查询</p>
          </div>
        </div>
      </div>
      <div class="o_service" v-show="serviceList.moreServeDatas && serviceList.moreServeDatas.length>0">
        <p class="c_p">更多服务</p>
        <div class="service_list">
          <div class='item' :index="index" key='index' v-for="(item,index) in serviceList.moreServeDatas">
              <router-link to="/serviceDetail" v-if="item.serveCode =='GLASSSUPPORT'">
                <div :class="'i_com i_'+item.serveCode">&nbsp;</div>
                <p>{{item.serveName}}</p>
              </router-link>
              <router-link :to="{path:'goods',query:{'serveCode':item.serveCode,'serviceInit':'init_10000'}}" v-else-if="item.items.length>1||(item.items.length==1 && item.items[0].productId=='')">
                <div :class="'i_com i_'+item.serveCode">&nbsp;</div>
                <p>{{item.serveName}}</p>
              </router-link>
              
              <router-link :to="{path:'details',query:{'pid':item.items[0].productId,'psource':1}}" v-else>
                <div :class="'i_com i_'+item.serveCode">&nbsp;</div>
                <p>{{item.serveName}}</p>
              </router-link>
          </div>
          <div class='item'>
            <router-link :to="{path:'goods',query:{'serveCode':'','serviceInit':'init_10000'}}">
              <div class="i_more">&nbsp;</div>
              <p>更多</p>
            </router-link>
          </div>
        </div>
      </div>


    </article>
    <!-- 申请会员提示 -->
    <div class="ck-pop-toast" v-if="toast3">
      <div class="inner">
        <div class="ck-pop">
          <div class="ck-pop-close" @click="toastClose"></div>
          <div class="ck-pop-bd">
            <p class="p1">亲，您不是会噢！</p>
            <div class="s-pop-btn" @click="applyGo">免费申请会员</div>
          </div>
        </div>
      </div>
    </div>
    <div class="question_fixed_btn"><router-link to="interactionHome" class="q_logo">&nbsp;</router-link></div>
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        title: '会员权益首页',
        imgBaseUrl: window.imgBaseUrl,
        toast3:false,
        isVip: window.isVip,
        openId:window.openId,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/serve/rightsHomeDatas?orgCode='+window.orgCode+'&shopsid='+window.shopsid+'&openId='+window.openId,
        apiActionUrl: window.baseUrl+window.appName+'/shopapi/func/product/displayedActivity?orgCode='+window.orgCode,
        serviceList: {},
        rechargeUrl:'',
        wxHomeUrl:'',
        CodeObj:window.CodeObj,
        loginUrl: 'http://leancare.net/lchat2/',
        orgCode:window.orgCode,
        userInfo:{}
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchHomeData();
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
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_SERVICEHOME});
        console.log(this.CodeObj,'this.CodeObj');
        //匹配用户信息跳转地址
        if(this.CodeObj[this.orgCode]){
          this.loginUrl = this.loginUrl + this.CodeObj[this.orgCode]+'/memberCenter/memberInfoByPassIndex?OpenId=' + window.openId;
        }else{
           this.loginUrl = "http://www.autoheader.com/shop/wxpay/shophome/"+this.orgCode+"?openId=" + window.openId;
        }
      },
      goTo (str){
        if(str == 'rechargeUrl'){
          if(window.serviceUrl && window.serviceUrl.rechargeUrl !=''){
            window.location.href = window.serviceUrl.rechargeUrl;
          }else{
            this.$router.push('/recharge');
          }
        }else if(str == 'violationQuery'){
          this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_SERVICEHOME_BTN_VIOLATIONQUERY});
          window.location.href = "https://ddz.io/9e8895?cc=1235.1001";
        }else{
          if(window.serviceUrl && window.serviceUrl.wxHomeUrl !=''){
            window.location.href = window.serviceUrl.wxHomeUrl;
          }else{
            this.$router.push('/home');
          }
        }
      },
      applyGo() {
        //跳去申请会员
        this.$router.push('/vipApply');
      },
      toastOpen(status){
        var _status = 'PAGE_SERVICEHOME_BTN_VIPCART_STATUS_'+ status;
        this.$store.dispatch('log', {account:LOGCODE.SERVICE[_status]});
        window.location.href=this.loginUrl;
      },
      toastClose(){
        this.toast3 = false;
      },
      fetchHomeData: function() {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
               that.serviceList = data.var;
               that.userInfo = data.var.userInfo;
               that.userInfo.couponCount = data.var.couponCount;
               setSessionStore('interaction_name_info',data.var.userInfo.userName);
                var dj = that.serviceList.moreServeDatas.filter(function(data){
                  return data.serveCode != 'DJSERVICES';
                });
                that.serviceList.moreServeDatas = dj;
                //判断logo地址是微信头像还是系统图片
                if(that.userInfo.userPic&&that.userInfo.userPic.split('http').length==1){
                  that.userInfo.userPic = that.imgBaseUrl + that.userInfo.userPic;
                }
                //判断代价服务在哪个服务列表咧
                var dj = that.serviceList.myServeDatas.filter(function(data){
                  return data.serveCode == 'DJSERVICES';
                });
                that.serviceList.isHasDj = dj.length>0?true:false;
                if(that.userInfo.phone){
                  localStorage.setItem('serviceInfoPhone', that.userInfo.phone);
                }
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              } 
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>
<style lang="less">
  @import "../../styles/common/service.less";
</style>