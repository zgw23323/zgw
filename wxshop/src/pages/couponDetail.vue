<template>
  <section class="ck-doc couponDetail">
     <article class="couponDetail_con" v-if="!tips1">
      <section class="top">{{couponDetail.couponName}}</section>
      <section class="center">
        <div>￥<span>{{couponDetail.couponAmount}}</span></div>
        <div class="coupon_user_btn" @click='couponBtn' v-if="isAdd">立即领取</div>
        <div class="coupon_user_btn" @click='goToUserCoupon' v-if="!isAdd">立即使用</div>
      </section>
      <ul>
        <li>满{{couponDetail.leastConsumpMoney}}可用</li>
        <li v-if="couponDetail.isShopCoupon==1 && couponDetail.isRechargeCoupon == 1"><em></em>适用部分商品、适用充值</li>
        <li v-else-if="couponDetail.isShopCoupon==1 && couponDetail.isRechargeCoupon==2"><em></em>部分商品可用</li>
        <li v-else-if="couponDetail.isShopCoupon==2 && couponDetail.isRechargeCoupon==1"><em></em>适用全部商品、适用充值</li>
        <li v-else-if="couponDetail.isShopCoupon==2 && couponDetail.isRechargeCoupon==2"><em></em>适用全部商品</li>
        <li v-else><em></em>适用充值</li>
        <li>活动时间:{{couponDetail.startTime}} ~ {{couponDetail.endTime}}</li>
      </ul>
    </article>
    <div class="tipsPage" v-if="tips1">
        <div class="tipsBox">
          <div class="tipStatus" data-type="busy"></div>
          <p>亲，网络出问题啦~~</p>
          <div class="btns" @click="reLoad">重新加载</div>
        </div>
    </div>
  </section>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,setSessionStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
export default { 
    data() {
      return {
          title: '优惠券详情',
          isAdd:true,
          couponDetail:{},
          tips1:false,
          couponId:'',
          //领取优惠券
          apiAddUserCoupon:window.baseUrl+window.appName+'/shopapi/func/coupon/addUserCoupon?orgCode='+ window.orgCode +'&shopsid='+ window.shopsid,
          apiCouponDetailUrl: window.baseUrl+window.appName+'/shopapi/func/coupon/wxgetCouponInfoById?orgCode='+window.orgCode       
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchCoupontDetail();
    },
    watch: {
      '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
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
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_COUPONDETAIL});
      },
      //去使用优惠券
      goToUserCoupon(){
          var _url = '';
          setSessionStore('couponId',this.couponId);
          if(this.isRechargeCoupon==1 && this.isShopCoupon == 0){ //跳去充值
              _url = '/recharge';
          }else{
              _url = '/goods';
          }
          this.$router.push({
              path: _url,
              query: {
                pId: 'coupon'
              }
          });
          
      },
      //立即领取优惠券
      couponBtn(str){
        let that = this;
          that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_COUPONDETAIL_COUPON_ADD});   
          that.$http.post(that.apiAddUserCoupon,{'couponId':this.couponId,'couponCount':1})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              // that.isAdd = false;
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_COUPONDETAIL_COUPON_ADD_OK});
              that.$dialog.notify({mes: '领取成功', icon: 'error', timeout: 2000,callback: () => {
                that.$router.push('/serviceHome');
              }});
            }else if(data.code == "USER_COUPON_ALREADY_RECEIVED") {
              // that.isAdd = false;
              that.$dialog.notify({mes: '您已经领取过了哦~', icon: 'error', timeout: 2000,callback: () => {
                 that.$router.push('/serviceHome');
              }});
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_COUPONDETAIL_COUPON_ADD_FAIL});
            }else if(data.code == "COMMON_ADD_FAIL") {
              that.isAdd = true;
              that.$dialog.notify({mes: '领取失败', icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_COUPONDETAIL_COUPON_ADD_FAIL});
            }else if(data.code == "PARAM_ERROR") {
              that.isAdd = true;
              that.$dialog.notify({mes: '参数错误', icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_COUPONDETAIL_COUPON_ADD_FAIL});
            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
              }});
            }else {
              that.isAdd = true;
              that.$dialog.notify({mes: '系统异常', icon: 'error', timeout: 1200});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      fetchCoupontDetail(){
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        var cou = window.location.href.split('#/')[0].split('couponId=')[1].split('&')[0];
        this.couponId = cou;
        // that.$http.post(that.apiCouponDetailUrl+'&couponId='+this.$route.query.couponId)
        that.$http.post(that.apiCouponDetailUrl+'&couponId='+cou)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
                that.couponDetail = data.var;
                that.couponDetail.startTime = that.couponDetail.startTime.split(' ')[0];
                that.couponDetail.endTime = that.couponDetail.endTime.split(' ')[0];
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
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
