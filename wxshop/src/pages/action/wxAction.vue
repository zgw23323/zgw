<template>
	<section class="ck-doc wxAction">
    <div class="action_content">
        <div class="action_time">活动时间：2019.9.1 - 2017.9.30</div>
        <!-- <div class="action_mess">会员充值活动，100%赠送三重大礼包，</br>“<span class="a_red">赠送金</span>”、“<span class="a_red">代金券</span>”、更有“<span class="a_red">现金红包</span>”等着您，</br><span class="a_red">100万</span>的活动奖励让您抢到手软~~</div> -->
        <div class="a_recharge" @click="goToRecharge">&nbsp;</div>
        <div class="action_recharge">
          <p class="p1">下面的“老司机”们已经得到超多奖励哦！难到您想错过吗？</p>
          <div id="a_outer">
            <ul id="ul1">
              <li v-for="(recharge,index) in rechargeList" @click="gun('stop')" @touchend="star">
                <div class="item1">
                  <div class="h_logo">
                    <!-- <img :src="imgBaseUrl+recharge.headshot" v-if="recharge.headshot!=''" /> -->
                    <img :src="recharge.headshot" v-if="recharge.headshot!=''" />
                    <img src="../../img/nav_icon_me_nor@3x.png" v-else />
                  </div>
                  <div>
                    <p class="p22">{{recharge.phone}}</p>
                    <p class="p24" v-if="(recharge.presented !=''&& parseFloat(recharge.presented)>0)||(recharge.voucher !='' && parseFloat(recharge.voucher)>0)||(recharge.redbagAmount !='' && parseFloat(recharge.redbagAmount)>0)">
                      获得<span v-if="recharge.presented !=''&& parseFloat(recharge.presented)>0"><span class="a_f">{{parseFloat(recharge.presented)}}</span>元赠送金</span>
                      <span v-if="recharge.voucher !='' && parseFloat(recharge.voucher)>0 && recharge.presented !='' && parseFloat(recharge.presented)>0">、<span class="a_f">{{parseFloat(recharge.voucher)}}</span>元代金券</span>
                      <span v-if="recharge.voucher !=''& parseFloat(recharge.voucher)>0 && (recharge.presented ==''|| !parseFloat(recharge.presented)>0)"><span class="a_f">{{parseFloat(recharge.voucher)}}</span>元代金券</span>
                      <span v-if="recharge.redbagAmount !='' && parseFloat(recharge.redbagAmount)>0 && recharge.resultCode=='SUCCESS' && ((recharge.voucher !=''&& parseFloat(recharge.voucher)>0)|| (recharge.presented !='' && parseFloat(recharge.presented)>0))">，还有<span class="a_f">{{parseFloat(recharge.redbagAmount)}}</span>现金红包</span>
                      <span v-if="recharge.redbagAmount != '' && parseFloat(recharge.redbagAmount)>0 && (recharge.presented == '' && parseFloat(recharge.presented)==0) && recharge.resultCode=='SUCCESS' && (recharge.voucher == ''|| parseFloat(recharge.voucher)==0)"><span class="a_f">{{parseFloat(recharge.redbagAmount)}}</span>现金红包</span>
                    </p>
                  </div>
                </div>
                <div class="item2">
                   <p class="p20">{{recharge.rechargetime}}</p>
                   <div class="p18"><i class="a_address">&nbsp;</i><span>{{recharge.storesname}}</span></div>
                </div>
              </li>
            </ul>
            <ul id="ul2"></ul>
          </div>
        </div>
    </div>
    
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        title: '有道十五载 感恩会员情',
        imgBaseUrl: window.imgBaseUrl,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/recharge/wxListRecharge',
        // apiUrl:"http://www.autoheader.com/shop/shopapi/func/recharge/wxListRecharge",
        rechargeList:[],
        timer:null,
        CodeObj:window.CodeObj,
        // rechargeUrl: window.baseUrl+window.appName +'/wxpay/pref?module=recharge&orgCode='
        rechargeUrl: 'http://leancare.net/lchat2/'
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      // this.gun('star');
      this.fetchData();
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
 
        this.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_WXACTION});
        var org = window.location.href.split('orgCode=')[1].split('&')[0];
        if(this.CodeObj[org]){
          this.rechargeUrl = this.rechargeUrl + this.CodeObj[org]+'-recharge/memberCenter/memberRedirectIndex?OpenId=' + window.openId;
        }else{
          // this.rechargeUrl = window.baseUrl+window.appName +'/wxpay/shophome/' +org + "?openId=" + window.openId;
          this.rechargeUrl = window.baseUrl+window.appName + '/wxpay/pref?openId='+window.openId+'&orgCode='+org +'&module=recharge'
        }
        console.log('orgCode=>',this.rechargeUrl);
      },
      goToRecharge(){
        this.$store.dispatch('log', {account:LOGCODE.ACTION.BTN_PAGE_ACTION_GO_RECHARGE});
        window.location.href = this.rechargeUrl;
      },
      star(){
        this.gun('stop');
        this.gun('star');
      },
      fetchData: function() {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl,that.opts)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
               that.rechargeList = data.var;
               for(var i=0;i<that.rechargeList.length;i++){
                 var t = that.rechargeList[i];
                 that.rechargeList[i].rechargetime = t.rechargetime.split(' ')[1].substring(0,5);
                 that.rechargeList[i].userName = t.userName.length>2?t.userName.substring(0,1)+'***'+t.userName.substring(t.userName.length-1):t.userName.substring(0,1)+'*';
                 if(t.headshot !=''){
                  var h = t.headshot.split('http');
                    if(h.length>1){
                      that.rechargeList[i].headshot = 'http'+ h[1];
                    }else{
                      that.rechargeList[i].headshot = that.imgBaseUrl + t.headshot;
                    }
                 }
                 if(t.phone){
                  that.rechargeList[i].phone = t.phone.substring(0,3)+'******'+ t.phone.substring(t.phone.length-2);
                 }
               }
               that.gun('star');
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
      },
      gun(str){
        var outer=document.getElementById('a_outer');
        var ul1=document.getElementById('ul1');
        var ul2=document.getElementById('ul2');
        // timer = null;
        if(outer.offsetHeight<=ul1.offsetHeight){
          ul2.innerHTML=ul1.innerHTML;
        } 
         
        if(str == 'stop'){
          clearInterval(this.timer);
          outer.style.overflow='scroll';
        }else{
          outer.removeAttribute('style','overflow:scroll');
          
          this.timer = setInterval(function(){
            outer.scrollTop++;
            if(outer.scrollTop>=ul1.offsetHeight){
              outer.scrollTop=0;
            }
           },50);
          console.log('star',this.timer);
        }
      }

    }
  }
</script>