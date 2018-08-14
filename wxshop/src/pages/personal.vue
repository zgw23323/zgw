<template>
  <section class="ck-doc personal">
    <div v-if="!tips1">
      <div class="personal-hd">
        <yd-cell-group>
          <yd-cell-item arrow href="/modify" type="link">
            <span slot="left">
              <div class="u-info">
                <div class="u-head" :style="{backgroundImage: 'url('+imgBaseUrl+personData.logopath + ')'}"></div>
                <div class="u-des">
                  <p class="u-name">
                    {{personData.userName}}
                    <i class="i-phone-2"></i>
                    <i class="i-vipcart" :class="{'light': personData.type == 1 ? true : false}"></i>
                  </p>
                  <P class="u-carType">{{personData.cartypeName == '' ? '请完善您的资料' : personData.cartypeName}}</P>
                </div>
              </div>
            </span>
          </yd-cell-item>
        </yd-cell-group>

        <yd-grids-group :rows="2">
          <yd-grids-item link="/obligations?type=2" >
            <yd-icon slot="icon" name="waitorder"></yd-icon>
            <span slot="text">待付款</span>
            <yd-badge slot="badge" type="danger" v-text="personData.unpaidOrders" v-if="show"></yd-badge>
          </yd-grids-item>
          <yd-grids-item link="/obligations?type=1">
            <yd-icon slot="icon" name="allorder"></yd-icon>
            <span slot="text">全部订单</span>
          </yd-grids-item>
        </yd-grids-group>
      </div>

      <div class="personal-bd">
        <yd-cell-group class="cell-group1" v-if="personData.type == 1">
          <yd-cell-item arrow :href="cardInfo.linkTo" type="link">
            <yd-icon slot="icon" name="vip"></yd-icon>
            <span slot="left" class="vip-des">我的会员卡</span>
            <span slot="right">{{cardInfo.cardNo}}</span>
          </yd-cell-item>
          <yd-cell-item>
            <span slot="left">我的余额</span>
            <span slot="right">￥{{parseFloat(personData.balance).toFixed(2)}}</span>
          </yd-cell-item>
          <yd-cell-item>
            <span slot="left">我的积分</span>
            <span slot="right">{{personData.integral}}</span>
          </yd-cell-item>
          <yd-cell-item arrow href="/coupon?mark=person" type="link" v-if="userCoupon>0">
            <yd-icon slot="icon" name="coupon"></yd-icon>
            <span slot="left" class="vip-des">我的优惠券</span>
            <span slot="right">{{userCoupon}}</span>
          </yd-cell-item>
          <yd-cell-item v-else>
            <yd-icon slot="icon" name="coupon"></yd-icon>
            <span slot="left" class="vip-des">我的优惠券</span>
            <span slot="right">无</span>
          </yd-cell-item>
        </yd-cell-group>

        <yd-cell-group class="cell-group2">
          <yd-cell-item arrow href="/address?mark=list" type="link">
            <yd-icon slot="icon" name="address"></yd-icon>
            <span slot="left" class="address-des">收货地址</span>
          </yd-cell-item>
        </yd-cell-group>

        <div class="personal-ft-btnswrap">
          <div v-if="showEsc" class="esc-btn" @click="esc">退出</div>
          <div v-if="personData.type == 0" class="apply-vip-btn" @click="applyGo">免费申请会员</div>
        </div>

        <!-- <div class="personal-ft-btnswrap"> -->
        <div class="clear-data" @click="clearData">清除缓存</div>
        <!-- </div> -->
      </div>
    </div>
    <!-- 清除缓存提示弹窗 -->
    <yd-popup v-model="show1" position="center" width="70%">
        <p class="clear-mess">
          您确定要清除缓存么？
        </p>
        <div class="clear-btn">
            <span @click="show1=false">取消</span>
            <span @click="clearData">确定</span>
        </div>
    </yd-popup>
    <div class="tipsPage" v-if="tips1">
      <div class="tipsBox">
        <div class="tipStatus" data-type="busy"></div>
        <p>亲，网络出问题啦~~</p>
        <div class="btns" @click="reLoad">重新加载</div>
      </div>
    </div>

    <ck-nav></ck-nav>
  </section>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,removeStore,setSessionStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
  export default {
    data() {
      return {
        tips1: false,
        show1: false,
        show: false,
        title: '个人中心',
        showEsc: false,
        userCoupon:0,
        imgBaseUrl: window.imgBaseUrl,
        apiPersonUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxUserPersonalCenter?orgCode='+window.orgCode+'&shopsid='+window.shopsid,
        apiUserCouponListUrl: window.baseUrl+window.appName+'/shopapi/func/coupon/userCouponList?orgCode='+window.orgCode+'&shopsid='+window.shopsid,
        apiLoginOutUrl: window.baseUrl + window.appName+'/loginapi/appdevice/loginout',
        optsPerson: {},
        opsCoupon:{
          curPage:1,
          pageSize:6,
          isRecharge:'',
          isShop:''
        },
        personData: {}
      }
    },
    watch: {
      '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchPersonList();
      
    },
    computed: {
      ...mapState([
        'headerTitle', 'news', 'cardInfo'
      ])
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
        this.showEsc = window.showEsc;

        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_PERSONAL});
      },
      applyGo() {
        //跳去申请会员
        this.$router.push('/register');
      },
      clearData() {
        sessionStorage.clear();//清除临时缓存
        localStorage.clear();//清除永久缓存
        localStorage.setItem("_orgCode", window.orgCode);

        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiLoginOutUrl,{}).then((response) => {
          const data = response.data;
          console.log('data:', data);
        });

        //跳去重定向页面
        if(window.redirectUrl == 'null' || window.redirectUrl == 'undefined' || window.redirectUrl == '') {
          this.$router.push('/home');
        }else {
          window.location.href = window.redirectUrl;
        }
      },
      esc() {
        //退出登录
        window.shopsid = "";
        //清除缓存
        removeStore('_shopsid');
        //localStorage.removeItem('_cartDetailsData');
        //localStorage.removeItem('_saveCart');

        //退出回到首页
        if(window.redirectUrl == 'null' || window.redirectUrl == 'undefined' || window.redirectUrl == '') {
          this.$router.push('/home');
        }else {
          window.location.href = window.redirectUrl;
        }
      },
      fetchPersonList() {
        //获取个人中心页面数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiPersonUrl,that.optsPerson)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "S_OK") {
              that.personData = data.var;
              if(that.personData.unpaidOrders != 0) {
                that.show = true;
              }
              //缓存详情数据
              // sessionStorage.setItem("_personData", JSON.stringify(that.personData));
              setSessionStore("_personData",that.personData);

              // 更新 vuex中的变量  type: 0普通，1会员
              //status  1:有卡，2:卡申请中，3:无卡
              if(that.personData.type == 1 && that.personData.status == 1){                
                this.$store.commit('UPDATE_CARDINFO', {'cardNo': that.personData.vipCard, 'linkTo':'/vipCard'});
              }              
              this.fetchUserCouponData();
            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else {
              that.tips1 = true;
              this.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
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
      //获取我的优惠券列表
      fetchUserCouponData(){
        //获取首页数据
        let that = this;
        that.$http.post(that.apiUserCouponListUrl,that.opsCoupon)
          .then((response) => {
            that.tips1 = false;
            let data = response.data;
            if(data.code == "S_OK") {
               that.userCoupon = data.var.totalCount;
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
      }
    }
  }
</script>
