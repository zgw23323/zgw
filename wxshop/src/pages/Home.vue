<template>

  <section class="ck-doc home">

    <div v-if="!tips1">
      <div class="home-banner">
        <yd-slider v-if="!isSingle" autoplay="3000" :ready="start1">
          <yd-slider-item :key="banner.id" v-for="banner in bannerList">
            <div @click="detailsGo(banner)" class="bannerimg" :style="{backgroundImage: 'url(' + imgBaseUrl+banner.logopath + ')'}" ></div>
          </yd-slider-item>
        </yd-slider>

        <div v-if="isSingle" @click="detailsGo(bannerList[0])" class="bannerimg" :style="{backgroundImage: 'url(' + imgBaseUrl+bannerList[0].logopath + ')'}"></div>
      </div>

      <!-- 改版新UIstar -->
      <div class="home_nav">
        <div class="n_item" v-for="item in sortList" @click="navBtn(item)">
          <router-link :to="{path: 'goods', query: {pId: item.id}}" class="me_right_com meu_right_top">
            <div><i :class="item.icon">&nbsp;</i></div>
            <span>{{item.name}}</span>
          </router-link>
        </div>
        <div class="n_item">
          <router-link :to="{path: 'goods', query: {pId: '-1'}}" class="me_right_com meu_right_top">
            <div><i class="MORE">&nbsp;</i></div>
            <span>更多服务</span>
          </router-link>
        </div>
      </div>
      <div class="home_action" v-if="optAction.secKillCount>0 || optAction.restrictCount>0">
        <div class="a_head">
          <div class="a_logo">&nbsp;</div>
          <div>
            <span class="t_mess" v-if="skillStatu.statu==3">距结束</span>
            <span class="t_mess" v-if="skillStatu.statu==1">距开始</span>
            <span class="t_mess" v-if="skillStatu.statu==4">已结束</span>
            <yd-countdown class="s1" :cursystem="skillStatu.currentTime" :callback="countDownCall" :time="skillStatu.endTime" :format="'{%h}:{%m}:{%s}'" v-if="skillStatu.statu==3"></yd-countdown>
            <yd-countdown class="s1" ref='countDown' :cursystem="skillStatu.currentTime" :callback="countDownCall" :time="skillStatu.startTime" :format="'{%h}:{%m}:{%s}'" v-if="skillStatu.statu==1"></yd-countdown>
          </div>
        </div>
        <ul class="action_product">
          <li v-for="item,index in skillProduct" @click="goTo(item.id,item.activityId,item.serveCode,2)" v-if="index<3">
            <div><img :src='imgBaseUrl+item.prodPic'/></div>
            <div class="d_1">{{item.prodTitle}}</div>
            <div class=" cur d_2" v-if="item.status==3">抢购中...</div>
            <div class="d_2" v-if="item.status==1">{{item.curDate}}开抢</div>
            <div class="d_2" v-if="item.status==4">已结束</div>
          </li>
        </ul>
        <div class="more_btn" v-if="skillProduct.length>3"><router-link :to="{path: 'skillProduct', query: {pId: 'secKillCount',navTitle:'秒杀'}}">更多</router-link></div>
      </div>
      <!-- end -->
      <!-- 领取优惠券入口 -->
      <div class="h-coupon" @click='goToCoupon' v-if="optAction.couponCount>0 && couponEnable.all && couponEnable.home">&nbsp;</div>
      <div class="h-coupon" @click='goToCoupon' style="display:none;">&nbsp;</div>
      <div class="goods-list" v-if="productList.length>0">
        <div class="h_title">
            <p>推荐商品</p>
            <p>每件商品都是精心挑选</p>
        </div>
        <ul class="good_product">
          <li :key="product.id" class='goods-par-dom' @click="goDetails(product.id,product.serveCode,product.prodSource)" v-for="product in productList">
            <!-- 权益服务商品 -->
            <div v-if="product.serveCode != ''&& product.prodSource==4 && !product.joinActivity">
              <div class="goodsimg imgLazy" slot="img" v-lazy:background-image="imgBaseUrl+product.prodPic"></div>
              <div class="s_1" v-if="product.serveType==1">{{product.prodName}} (无限次)</div>
              <div class="s_1" v-else>{{product.prodName}} ({{product.serveCount+'次'}})</div>
              <p class="goods-price">
                <span class="service_price_mess">￥{{product.memberPrice}}<span class="list-del-price">￥{{product.orgPrice}}</span></span>
                <span class="s_memberMess">积分兑现金</span>
              </p>
            </div>
            <!-- 普通商品 -->
            <div v-else>
              <div class="goodsimg imgLazy" slot="img" v-lazy:background-image="imgBaseUrl+product.prodPic"></div>
              <div class="s_1">{{product.prodTitle}}</div>
              <p class="goods-price">
               <span class="service_price_mess">￥{{product.memberPrice}}<span class="list-del-price">￥{{product.orgPrice}}</span></span>
                <span class="s_memberMess" v-if="product.springProdId">一元领取代金券</span>
                <span class="s_memberMess" v-else>积分兑现金</span>
              </p>
            </div>
          </li>
        </ul>
        <div class="more_btn"><router-link :to="{path: 'goods', query: {pId: '-1'}}">更多</router-link></div>
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

    <ck-nav></ck-nav>
  </section>

</template>

<script type="text/babel">
  document.title = '友道会商城';
  import {setShopsId,setStore,getStore,setSessionStore,getSessionStore} from '../utils/assist';
  import {codeKeyValue} from '../utils/errorCode';
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
        apiSortUrl: window.baseUrl+window.appName+'/shopapi/func/sort/findMenu?orgCode='+window.orgCode,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/product/homePageList?orgCode='+window.orgCode,
        apiActionUrl: window.baseUrl+window.appName+'/shopapi/func/product/displayedActivity?orgCode='+window.orgCode,
        apikillActionUrl: window.baseUrl+window.appName+'/shopapi/func/product/queryProductList?orgCode='+window.orgCode+"&fromSource=2",
        apiRestrictActionUrl: window.baseUrl+window.appName+'/shopapi/func/product/queryProductList?orgCode='+window.orgCode+"&fromSource=1",
        opts: {curPage: 1, pageSize: 8, isBanner: 1},
        bannerList: [],
        opt:{
          curPage:1,
          pageSize:6,
          identifierCode:2,
          name:'秒杀'
        },
        isCoundownCall:false,
        optAction: {},//活动信息对象
        sortList:[],//分类列表
        skillProduct:[],//秒杀商品
        skillStatu: {},//秒杀的状态
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
      this.fetchHomeData();
      this.getActionFun();
      this.getSortFun();
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
        console.log(1111);
        $('.event_content').animate({
          top:'24%',
        },1000,'linear');
      },
      goToCoupon(){
         this.$router.push('/coupon?mark=list');
      },
      //分类点击
      navBtn(pItem){
        if(pItem.linkurl && pItem.linkurl.split('http').length>1){
          window.location.href = pItem.linkurl;
        }else if(pItem.linkurl && pItem.linkurl != ''){
          this.$router.push({path: pItem.linkurl});
        }else{
          this.$router.push({
            path: '/goods',
            query: {pId: pItem.id}
          });
        }
      },
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_HOME});
      },
      countDownCall(){
        if(this.skillStatu.statu==3){
          this.skillStatu.statu=4;
        }else if(this.skillStatu.statu==1 && this.skillStatu.startTime !=''){
          this.isCoundownCall = true;
          ajaxModel.queryProductList(this,this.fectchQueryFun,{curPage:1,pageSize:6,identifierCode:2,name:'秒杀'},2);
          ajaxModel.queryProductList(this,this.fectchQueryFun,{curPage:1,pageSize:6,identifierCode:3,name:'折扣'},3);
        }
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
          this.goDetails(opt.productId, opt.serveCode, (opt.prodSource) || "4");
        }
      },
      //秒杀商品跳去详情页
      goTo(id,activityId,serveCode,prodSource) {
          //跳去详情页
          // if(serveCode == ''){
            this.$router.push({
              path: '/details',
              query: {
                pid: id,
                psource:prodSource,
                activityId:activityId
              }
            });
          // }
          // else{
          //   var str = serveCode.split('_')[0];
          //   this.$router.push({
          //     path: '/serviceDetail',
          //     query: {
          //       pid: id,
          //       serveCode:str,
          //       activityId:activityId,
          //       psource:this.source
          //     }
          //   });
          // }
          
      },
      goDetails(id,serveCode,prodSource) {
          //跳去详情页
          if(serveCode && serveCode.split('_')[0] =='GLASSSUPPORT'){
            this.$router.push({path: '/serviceDetail'});
          }else{
            this.$router.push({
              path: '/details',
              query: { 
                pid: id
              }
            });
          }
      },
      fetchHomeData: function() {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl,that.opts)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "S_OK") {
              let productArr = [];
              let bannerArr = [];
              for (let plist of data.var.productList) {
                productArr.push(plist)
              }
              for (let blist of data.var.bannerList) {
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
      },
      fectchQueryFun(result){
        this.skillProduct.push(...result);
        var _statu = this.skillProduct.filter((data)=>{
          return data.status == 3;//抢购中
        });
        if(_statu.length){
          this.skillStatu.statu = 3;
          this.skillStatu.currentTime =_statu[0].currentTime;
          this.skillStatu.endTime = _statu[0].endTime;
        }else{
          _statu = this.skillProduct.filter((data)=>{
            return data.status == 1;//未开始、预约中
          });
          if(_statu.length){
            this.skillStatu.statu = 1;
            this.skillStatu.startTime = _statu[0].startTime;
            this.skillStatu.endTime = _statu[0].endTime;
            this.skillStatu.currentTime =_statu[0].currentTime;
            if(this.isCoundownCall){
              this.$refs.countDown.run({acEndTime:this.skillStatu.endTime,currTime:this.skillStatu.currentTime});
            }
          }else{ 
            _statu = this.skillProduct.filter((data)=>{
              return data.status == 4;//已结束
            });
            if(_statu.length){
              this.skillStatu.statu = 4;
            }
          }
        }
        //循环是为了设置时间
        for(var i=0;i<this.skillProduct.length;i++){
          var item = this.skillProduct[i];
          if(item.status==1){
            var curDate = item.startTime !=''? item.startTime.split(' ')[1].split(':') : '';
            var _m =curDate !=''?Number.parseInt(curDate[0]):'';
            var _d = curDate !=''?Number.parseInt(curDate[1]):'';
            this.skillProduct[i].curDate = _m+'.'+_d;
          }else{
            this.skillProduct[i].curDate = '';
          }
        }
      },
      getActionFun(){
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiActionUrl)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "S_OK") {
              that.optAction = data.var;
              if(that.optAction.secKillCount>0){
                ajaxModel.queryProductList(this,this.fectchQueryFun,{curPage:1,pageSize:6,identifierCode:2,name:'折扣'},2);
              }
              if(that.optAction.restrictCount>0){
                ajaxModel.queryProductList(this,this.fectchQueryFun,{curPage:1,pageSize:6,identifierCode:3,name:'折扣'},3);
              } 
              if(that.optAction.events && that.optAction.events.length>0){
                //判断活动弹框是否有弹出过
                let _events = getSessionStore('events_is_open',false);
                if(typeof(_events) == 'boolean'){//等于说明找不到缓存数据、默认为是首次打开,否则不做任何处理
                  that.tips2=true;
                  setSessionStore('events_is_open','true');
                }
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
      },
      getSortFun(){
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiSortUrl)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "S_OK") {
              that.sortList = data.var.sort;
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
