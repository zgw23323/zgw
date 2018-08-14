<template>

  <section class="ck-doc home">

    <div v-if="!tips1">
      <div class="home-banner">
        <yd-slider v-if="!isSingle" autoplay="3000" :ready="start1">
          <yd-slider-item :key="banner.id" v-for="banner in bannerList">
            <div @click="detailsGo(banner)" class="bannerimg" :style="{backgroundImage: 'url(' + imgBaseUrl+banner.logopath + ')'}"></div>
          </yd-slider-item>
        </yd-slider>

        <div v-if="isSingle" @click="detailsGo(bannerList[0])" class="bannerimg" :style="{backgroundImage: 'url(' + imgBaseUrl+bannerList[0].logopath + ')'}"></div>
      </div>

      <div class="ck-search home-search-bar">
        <router-link to="/search">
          <div class="inner">
            <i class="iconfont">&#xe602;</i>
            <span>请输入搜索的商品名</span>
          </div>
        </router-link>
      </div>
      <div class='action-nav-meu'>
        <div class='nav-meu-left'>
          <!-- 最新商品 -->
            <router-link :to="{path: 'goods', query: {pId: '004',navTitle:'最新商品'}}" v-if="optAction.restrictCount==0">
              <div class="ac_commen_logo ac_new_logo">&nbsp;</div>
              <p>最新商品，给你新鲜</p>
              <div class= "ac_commen_img ac_new_img">&nbsp;</div>
            </router-link to="/goods">
          <!-- 限时购 -->
            <router-link :to="{path: 'goods', query: {pId: 'restrictCount',navTitle:'限时折扣'}}" v-if="optAction.restrictCount>0" >
              <div class="ac_commen_logo ac_buy_logo">&nbsp;</div>
              <p>优惠商品限时限量抢购</p>
              <div class= "ac_buy_img">&nbsp;</div>
            </router-link to="/goods">
        </div>
        <div class='nav-meu-right'>
          <router-link :to="{path: 'goods', query: {pId: 'secKillCount',navTitle:'秒杀'}}" class="me_right_com meu_right_top" v-if="optAction.secKillCount >0">
            <div>
              <!-- <p class="meu_new_title">新人专享</p> -->
              <p class="meu_miao_title">&nbsp;</p>
              <p class="p1">限量秒杀，一降到底</p>
            </div>
            <div class= "new_miao_img">&nbsp;</div>  
          </router-link>
          <router-link :to="{path: 'goods', query: {pId: '006',navTitle:'新人专享'}}" class="me_right_com meu_right_top" v-if="optAction.secKillCount==0">
            <div>
              <p class="meu_new_title">新人专享</p>
              <p class="p1">新人优惠，绝对低价</p>
            </div>
            <div class= "new_peo_img">&nbsp;</div>  
          </router-link>
          <router-link :to="{path: 'goods', query: {pId: '002',navTitle:'会员专享'}}" class="me_right_com">
            <div>
              <p class="meu_vip_title">会员专享</p>
              <p class="p2">最新优惠，时时享受</p>
            </div>
            <div class= "vip_peo_img">&nbsp;</div>
          </router-link>
        </div>
      </div>
      <!-- 领取优惠券入口 -->
      <div class="h-coupon" @click='goToCoupon' v-if="optAction.couponCount>0 && couponEnable.all && couponEnable.home">&nbsp;</div>
      <div class="h-coupon" @click='goToCoupon' style="display:none;">&nbsp;</div>
      <div class="goods-list" v-if="productList.length>0">
         <!-- 推荐商品 -->
        <h2 class="title">
          <router-link :to="{path: 'goods', query: {pId: '003',navTitle:'推荐商品'}}"><img class="goods-recommend-logo" src="../img/icon-recommend-logo@3x.png" /></router-link>
        </h2>
       <!--  <h2 class="title">
          <router-link :to="{path: 'goods', query: {pId: '003',navTitle:'推荐商品'}}"><img style="width:100%;" src="../img/actionImg/tui_head.png" /></router-link>
        </h2> -->
        <yd-list theme="3">
          <div :key="product.id" class='goods-par-dom' @click="goDetails(product.id,product.serveCode,product.prodSource)" v-for="product in productList">
            <!-- 权益服务商品 -->
            <yd-list-item v-if="product.serveCode != ''&& product.prodSource==4 && !product.joinActivity">
              <div class="goodsimg imgLazy" slot="img" v-lazy:background-image="imgBaseUrl+product.prodPic"></div>
              <span slot="title" v-if="product.serveType==1">{{product.prodName}} (无限次)</span>
              <span slot="title" v-else>{{product.prodName}} ({{product.serveCount+'次'}})</span>
              <yd-list-other slot="other">
                  <p class="goods-price">
                    <span class="service_price_mess">￥{{product.memberPrice}}<span class='s_memberMess'>活动价</span></span>
                    <span class="list-del-price">市场价￥{{product.orgPrice}}</span>
                  </p>
              </yd-list-other>
            </yd-list-item>
            <!-- 普通商品 -->
            <yd-list-item v-else>
              <div class="goodsimg imgLazy" slot="img" v-lazy:background-image="imgBaseUrl+product.prodPic"></div>
              <span slot="title">{{product.prodTitle}}</span>
              <yd-list-other slot="other">
                  <p class="goods-price">￥{{product.memberPrice}}<span class="list-del-price">￥{{product.orgPrice}}</span></p>
                  <span class="goods-memberPoint" v-if="product.springProdId">一元领取代金券</span>
                  <span class="goods-memberPoint" v-else-if="product.memberPrice-product.pointPrice>0">{{product.memberPoint}}积分可兑换{{product.memberPrice-product.pointPrice}}元现金</span>
              </yd-list-other>
            </yd-list-item>
          </div>
        </yd-list>
      </div>
    </div>

    <div class="tipsPage" v-if="tips1">
      <div class="tipsBox">
        <div class="tipStatus" data-type="busy"></div>
        <p>亲，网络出问题啦~~</p>
        <div class="btns" @click="reLoad">重新加载</div>
      </div>
    </div>

     <!--活动图片弹窗-->
    <yd-popup v-model="tips2" v-if="tips2" position="center" class="ser-legalfield" style="height:460px;" width="90%">
        <div class="event_content">
          <i class="e_tip_close" @click="tips2 = !tips2;">&nbsp;</i>
          <a :href="item.describe" class="event_tip" v-for="item in optAction.events">
              <img :src="imgBaseUrl+'/shop/downLoad/showLogo/'+item.pid+'?type=thumb'">
          </a>
        </div>
    </yd-popup>

    <!-- <yd-popup v-model="tips3" v-if="tips3" position="center" class="ser-legalfield"  width="81.333%">
        <div class="event_content">
          <router-link :to="{path:'actionProduct'}"><img src="../img/actionImg/action_tip_img.png" style="width: 100%;" /></router-link>
        </div>
    </yd-popup> -->

    <!-- <div class="ck-pop-toast" v-if="tips3" @click="tips3=false">
      <div class="event_content">
        <router-link :to="{path:'actionProduct'}"><img src="../img/actionImg/action_tip_img.png" style="width: 100%;" /></router-link>
      </div>
    </div> -->

    <ck-nav></ck-nav>
  </section>

</template>

<script type="text/babel">
  document.title = '友道会商城';
  import {setShopsId,setStore,getStore,setSessionStore,getSessionStore} from '../utils/assist';
  import {LOGCODE} from '../utils/logCode';
  import ajaxModel from '../utils/ajaxModel';
  export default {
    data() {
      return {
        ok: false,
        tips1: false,
        tips2:false,
        tips3: false,
        isSingle: false,
        start1: false,
        title: '友道会商城',
        couponEnable:window.couponEnable,
        imgBaseUrl: window.imgBaseUrl,
        opts: {curPage: 1, pageSize: 8, isBanner: 1},
        bannerList: [],
        optAction: {},//活动信息对象
        productList: []
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      console.log(window.couponEnable);
      //挂载完毕初始化页面数据
      this.markInit();
      ajaxModel.homePageList(this,this.fetchHomeData,this.opts);
      ajaxModel.displayedActivity(this,this.getActionFun);
      let _events = getStore('spring_is_open',false);
      if(typeof(_events) == 'boolean'){//等于说明找不到缓存数据、默认为是首次打开,否则不做任何处理
        this.run();
        setStore('spring_is_open','true');
      }else{
        this.tips3 = false;
      }
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
      run() {
        $('.event_content').animate({
          top:'24%',
        },1000,'linear');
      },
      goToCoupon(){
         this.$router.push('/coupon?mark=list');
      },
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_HOME});
      },
      detailsGo(opt) {
        //跳去商品详情
        if(opt.type==2){
          var orgCodeReg = /\{orgCode\}/;
          var openIdReg = /\{openId\}/;
          opt.linkUrl = opt.linkUrl.replace(orgCodeReg, window.orgCode);
          opt.linkUrl = opt.linkUrl.replace(openIdReg, window.openId);
          window.location.href = opt.linkUrl;
        }else if(!opt.serveCode){
          this.$router.push({ path: 'details', query: { 'pid': opt.productId}});
        }else{
          //this.$router.push({ path: 'details', query: { 'pid': opt.productId,'psource':1,'serveCode':opt.serveCode}});
          this.goDetails(opt.productId, opt.serveCode, (opt.prodSource) || "4");
        }
      },
      goDetails(id,serveCode,prodSource) {
          //跳去详情页
          if(serveCode == '' || prodSource !='4'){
            this.$router.push({
              path: '/details',
              query: { 
                pid: id
              }
            });
          }else{
            var str = serveCode.split('_')[0];
            this.$router.push({
              path: '/serviceDetail',
              query: {
                'pid': id,
                'serveCode':str,
                'psource':1
              }
            });
          }
      },
      fetchHomeData: function(result) {
        //获取首页数据
        let that = this;
        that.tips1 = false;
        let productArr = [];
        let bannerArr = [];
        for (let plist of result.productList) {
          productArr.push(plist)
        }
        for (let blist of result.bannerList) {
          bannerArr.push(blist)
        }
        that.productList = productArr;
        if(bannerArr.length != 0) {
          that.bannerList = bannerArr;
          if(bannerArr.length == 1) {
            that.isSingle = true;
          }else {
            that.start1 = true;
          }
        }
      },
      getActionFun(result){
        //获取首页数据
        let that = this;
        that.optAction = result;
        if(that.optAction.events && that.optAction.events.length>0){
          //判断活动弹框是否有弹出过
          let _events = getSessionStore('events_is_open',false);
          if(typeof(_events) == 'boolean'){//等于说明找不到缓存数据、默认为是首次打开,否则不做任何处理
            that.tips2=true;
            setSessionStore('events_is_open','true');
          }
        }
      } 
    },
    beforeDestroy: function () {
      console.log("beforeDestroy11" + this.$el);
      console.log('beforeDestroy',document.documentElement.scrollTop,document.getElementsByClassName('action-nav-meu'));
    }
  }
</script>

<style lang="less">
  @import "../styles/common/chokui.less";
  @import "../styles/common/style.less";
  .ck-pop-toast {
    background-color: rgba(0,0,0,.6)!important;
  }
  .event_content {
    // background-color: #333;
    width: 81.333%;
    position:fixed;
    left:9%;
    top:-50%;
  }
</style>
