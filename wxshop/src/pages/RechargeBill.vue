<template>
  <transition name="page">
    <section class="ck-doc rechargeDetail_info">
    <div class='rechargeDetail' v-if="!tips1">
        <div class="recharge-success-wrap">
          <div class="recharge-success-hd">
             <div class="r_money">{{parseFloat(billList.payMoney).toFixed(2)}}</div>
             <div class="r_statu">充值成功</div>
          </div>
          <div class="recharge-success-give">
             <div class="r_mess">恭喜您，本次充值获得</div>
             <div class="r_song">
                <div class="item" v-if="billList.giveMoney>0">
                  <p class="g_money">{{billList.giveMoney}}</p>
                  <p class="g_mess">赠送金</p>
                </div>
                <div class="item" v-if="billList.couponAmount>0">
                  <p class="g_money">{{billList.couponAmount}}</p>
                  <p class="g_mess">代金券</p>
                  <div class="g_btn" @click="fecthCouponList">查看</div>
                </div>
                <div class="item" v-if="billList.redbagAmount>0">
                  <p class="g_bao">&nbsp;</p>
                  <p class="g_mess">现金红包</p>
                </div>
             </div>
          </div>
          <div class="r_bd">
              <ul>
                <li>订单详情</li>
                <li><span>订单编号：</span>&nbsp;<span>{{billList.orderCode}}</span></li>
                <li><span>会<em class="w_2">&nbsp;</em>员<em class="w_2">&nbsp;</em>卡：</span>&nbsp;<span>{{billList.vipCard}}</span></li>
                <li><span>姓<em class="w_1">&nbsp;</em>名：</span>&nbsp;<span>{{billList.userName}}</span></li>
                <li><span>充值金额：</span><span>￥{{parseFloat(billList.payMoney).toFixed(2)}}</span></li>
                <li><span>赠送金额：</span><span>￥{{parseFloat(billList.giveMoney).toFixed(2)}}</span></li>
                <li><span>到账金额：</span><span>￥{{parseFloat(billList.arrivalMoney).toFixed(2)}}</span></li>
                <li><span>总<em class="w_2">&nbsp;</em>金<em class="w_2">&nbsp;</em>额：</span><span>￥{{parseFloat(billList.totalMoney).toFixed(2)}}</span></li>
                <li><span>下单时间：</span>&nbsp;<span>{{billList.createTime}}</span></li>
              </ul>
          </div>

        </div>
        
        <div class="tipsPage" v-if="tips1">
          <div class="tipsBox">
            <div class="tipStatus" data-type="busy"></div>
            <p>亲，网络出问题啦~~</p>
            <div class="btns" @click="reLoad">重新加载</div>
          </div>
        </div>
    </div>
    <div class="r_border" v-if="!tips1">&nbsp;</div>
    <div class="tipCoupon" v-if="show1" @click="closeTip">&nbsp;</div>
    <!-- <yd-popup v-model="show1" position="center" width="85.3333%"> -->
    <div class="tipConponCon" v-if="show1">
      <div class="p_head">
          <span>代金券</span>
          <span class="p_close" @click="closeTip">&nbsp;</span>
      </div>
      <ul class="g_coupon">
        <li class="g_cou1" v-for="item in couponList">
          <div class="item1">
            <div><span class="c_2">￥</span><span class="c_1">{{item.couponAmount}}</span></div>
            <div class="c_3">{{item.couponName}}</div>
            <div class="yu_top">&nbsp;</div>
            <div class="yu_bottom">&nbsp;</div>
          </div>
          <div class="item2" @click="goToUserCoupon(item)">
            <p class="go_mess">去购买</p>
            <div class="go_btn">&nbsp;</div>
          </div>
        </li>
         
      </ul>
    <!-- </yd-popup> -->
    </div>

      <div class="tipsPage" v-if="tips1">
        <div class="tipsBox">
          <div class="tipStatus" data-type="busy"></div>
          <p>亲，网络出问题啦~~</p>
          <div class="btns" @click="reLoad">重新加载</div>
        </div>
      </div>
    </section>
  </transition>

</template>

<script>
import {setShopsId} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
  export default {
    data() {
      return {
        show: true,
        tips1: false,
        show1:false,
        title: '会员卡充值',
        billList: {payMoney:0},
        aipBillList: window.baseUrl+window.appName+'/shopapi/func/recharge/wxFindRechargeRecord?orgCode='+window.orgCode,
        aipCouponList: window.baseUrl+window.appName+'/shopapi/func/recharge/wxListGiveCoupon?orgCode='+window.orgCode,
        optsBillList: {},
        couponList: []
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
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_RECARGEBILL});
      },
      openTip(){
        this.fecthCouponList();
      },
      closeTip(){
        this.show1 = false;
        document.getElementsByTagName('body')[0].style.overflow='auto';
      },
      //去使用优惠券
      goToUserCoupon(opt){
        document.getElementsByTagName('body')[0].style.overflow='auto';
        if(opt.productIds.length>1){ //跳去商品列表
          this.$router.push({
            path: 'goods',
            query: {
              pId: 'coupon'
            }
          });
          sessionStorage.setItem('couponId',opt.couponId);
        }else{//跳去商品详情
          this.$router.push({
            path: '/serviceDetail',
            query: {
              'pid': opt.productIds[0],
              'serveCode':opt.serveCode.split('_')[0],
              // 'serveCode':'DJSERVICES',
              'psource':1
            }
          });
        } 
      },
      fecthCouponList(){
        //获取代金券数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.optsBillList.id = that.$route.query.id;
        // that.$http.post(this.aipCouponList+'&shopsid='+window.shopsid,that.optsBillList)
        that.$http.post(this.aipCouponList+'&shopsid='+window.shopsid,{orderNumber:that.billList.orderCode})
          .then((response) => {
              const data = response.data;
              if(data.code == "S_OK") {
                that.couponList = data.var;
                this.show1 = true;
                document.getElementsByTagName('body')[0].style.overflow='hidden';
              }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                    this.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '您还没有登录', icon: 'error', timeout: 1200, callback: () => {
                    this.$router.push('/login');
                }});
              }else {
                that.tips1 = true;
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }
              setTimeout(()=>{
                this.$dialog.loading.close();
              },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);
        });
      },
      fetchBillList() {
        //获取充值结果详情数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.optsBillList.id = that.$route.query.id;
        that.$http.post(this.aipBillList+'&shopsid='+window.shopsid,that.optsBillList)
          .then((response) => {
          const data = response.data;
              that.tips1 = false;
              if(data.code == "S_OK") {
                that.billList = data.var;
              }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                    this.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '您还没有登录', icon: 'error', timeout: 1200, callback: () => {
                    this.$router.push('/login');
                }});
              }else {
                that.tips1 = true;
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }
              setTimeout(()=>{
                this.$dialog.loading.close();
              },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);
        });
      }
    }
  }
</script>