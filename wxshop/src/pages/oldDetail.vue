<template>
    <section class="ck-doc details">
      <div class="inner" v-if="!tips1">
        <div class="details-hd">
          <div class="details-banner">
            <yd-slider v-if="!isSingle" autoplay="3000" :ready="start1">
              <yd-slider-item :key="banner.id" v-for="banner in detailsData.picList">
                <div class="bannerimg" :style="{backgroundImage: 'url(' + imgBaseUrl+banner.prodPicId + ')'}"></div>
              </yd-slider-item>
            </yd-slider>
            <div v-if="isSingle" class="bannerimg" :style="{backgroundImage: 'url(' + imgBaseUrl+detailsData.picList[0].prodPicId + ')'}"></div>
          </div>
          <!-- 限时折扣 -->
          <div class='details-action' v-if="detailsData.restrict">
            <!-- 抢购中、即将开始 -->
            <div class="details-action-time action-common" v-if="actionStatu==1">
                <div class="de_ac_left"><span class="le_lo">&nbsp;</span><span class='le_mes'>抢购中</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acEndTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <div class="details-action-time action-common" v-if="actionStatu==3">
                <div class="de_ac_left"><span class="le_lo">&nbsp;</span><span class='le_mes'>即将开始</span></div>
                <div class="de_ac_right">
                   <p>距开始时间</p>
                   <yd-countdown class="s1" ref='countDown' :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acStarTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <!-- 活动已结束 -->
            <div class="details-action-over action-common" v-if="actionStatu==2">
                <div class="de_ac_left"><span class="le_lo">&nbsp;</span><span class='le_mes'>活动已结束</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="'0'" :countType="'1'" :time="'0'"  format=" {%h}:{%m}:{%s}"></yd-countdown>
                </div>
            </div>
          </div>
          <!-- 秒杀-->
          <div class='details-action' v-if="detailsData.skipe">
            <!-- 抢购中、即将开始 -->
            <div class="details-action-miao action-common" v-if="actionStatu==1">
                <div class="de_ac_left"><span class="le_lo_m">&nbsp;</span><span class='le_mes'>秒杀中</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acEndTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <div class="details-action-miao action-common" v-if="actionStatu==3">
                <div class="de_ac_left"><span class="le_lo_m">&nbsp;</span><span class='le_mes'>预约中</span></div>
                <div class="de_ac_right">
                   <p>距开始时间</p>
                   <yd-countdown class="s1" ref='countDown' :cursystem="optAction.currTime" :countType="'1'" :time="optAction.acStarTime" :callback="countDownCall" :format="optAction.acformat"></yd-countdown>
                </div>
            </div>
            <!-- 活动已结束 -->
            <div class="details-action-over action-common" v-if="actionStatu==2">
                <div class="de_ac_left"><span class="le_lo_m_over">&nbsp;</span><span class='le_mes'>活动已结束</span></div>
                <div class="de_ac_right">
                   <p>距结束时间</p>
                   <yd-countdown class="s1" :cursystem="'0'" :countType="'1'" :time="'0'"  format=" {%h}:{%m}:{%s}"></yd-countdown>
                </div>
            </div>
          </div>
          <div class="details-mes">
            <div class="details-title" v-text="detailsData.prodTitle"></div>
            <div class="details-other">
              <div class="details-price">
                <!-- 秒杀活动未开始 -->
                <span v-if="detailsData.skipe==true && actionStatu != 2">￥{{detailsData.skipePrice}}</span>
                <span v-else-if="detailsData.restrict==true && actionStatu != 2">￥{{detailsData.discountPrice}}</span>
                <span v-else>￥{{detailsData.memberPrice}}</span>
                <span class="details-del-price">￥{{detailsData.orgPrice}}</span>
              <!-- 秒杀活动未开始 -->
                <span class='action-appointment' v-if='detailsData.skipe==true && actionStatu == 3'  @click='reserveBtn(detailsData.activityId)'>立即预约</span>
              </div>
              <!-- 进度条 -->
              <div class="progress" v-if='(detailsData.skipe || detailsData.restrict) && actionStatu != 3'>
                  <div class="progress-bar" v-if="source == 2">
                      <span class="sr-only" :style="{'width': (detailsData.saleTotal/(detailsData.skipeStore+detailsData.saleTotal)*100).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span>
                  </div>
                   <div class="progress-bar" v-if="source == 3">
                     <!--  <span class="sr-only" :style="{'width': (detailsData.saleTotal/detailsData.stocksCount*100).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span> -->
                       <span v-if="!detailsData.initStocks" class="sr-only" :style="{'width': (100-detailsData.stocksCount).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span>
                      <span v-else-if="detailsData.stocksCount>0" class="sr-only" :style="{'width': (detailsData.initStocks-detailsData.stocksCount).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span>
                      <span v-else class="sr-only" :style="{'width': '100%','max-width': '100%'}">&nbsp;</span>
                  </div>
                  <div class="progress-count" v-if='source == 2'>剩余{{detailsData.skipeStore}}</div>
                  <div class="progress-count" v-if='source == 3'>剩余{{detailsData.stocksCount}}</div>
              </div>
            </div>
          </div>
        </div>
        <div class="details-action-menu">
          <p class="details-memberPoint" v-if="detailsData.memberPrice-detailsData.pointPrice>0 && detailsData.skipe==false && detailsData.restrict == false">
              <span class="action-mess">促销活动</span>
              <span>{{detailsData.memberPoint}}积分可兑换{{totalCurPrice}}元现金</span>
          </p>
          <!-- 秒杀中、秒杀结束限购提示 -->
          <p class="action-appointment-ing" v-if="detailsData.skipe==true && actionStatu != 3 && detailsData.restrictType ==2"><span class="action-mess no-mess">限购</span><span>每人限购{{detailsData.restrictCount}}件，超出不再享受秒杀价</span></p>
          <!-- 秒杀未开始预约提示 -->
          <div class='coupon-item action-miao' v-if="detailsData.skipe==true && actionStatu == 3">
             <span class="action-song no-r" >秒杀</span>
             <span>预计{{detailsData.startTime}}开始，请提前预约</span>
          </div>
           <!-- 领取优惠券 -->
          <div class='coupon-item' v-if="couponList.length>0 && couponEnable.all && couponEnable.detail"  @click='openTip(4)'>
             <span class="icon-coupon" >&nbsp;</span>
             <span class="">领取优惠券</span>
             <span class="right-btn">&nbsp;</span>
          </div>

          <div class='coupon-item i_flex_betwent' v-if="detailsData.springProdId">
            <div>
              <span class="icon-coupon" >&nbsp;</span>
              <span class="">一元领取代金券</span>
            </div>
            <router-link class="get_coupon" :to="{path:'actionProductDetail',query:{pid:detailsData.prodId,serveCode:'DJSERVICES'}}">领取</router-link>
          </div>
          <!-- 满减送详情 -->
  
          <div class='coupon-item' @click='getAppListReduced(3)' v-if="reducedInfo !=null">
             <span class="action-song" >满减送</span>
             <span class="">查看详情</span>
             <span class="right-btn">&nbsp;</span>
          </div>
          
        </div>
        <div class="details-bd">
          <h2 class="title">商品详情</h2>
          <div class="details-des" v-html="detailsData.prodContent"></div>
        </div>

        <div class="details-ft">
          <div class="buy-bar">
            <div class="buy-icon" @click="cartGo">
              <i class="iconfont">&#xe600;<yd-badge type="danger" v-text="goodsCount" v-if="show"></yd-badge></i>
            </div>
            <!-- <div class="buy-btns" v-if="detailsData.restrict && actionStatu !=1">
              <div class="buy-cart disabled">加入购物车</div>
              <div class="buybuybuy disabled">立即购买</div>
            </div> -->
            <div class="buy-btns">
              <div class="buy-cart disabled" v-if="(detailsData.skipe || detailsData.restrict) && actionStatu == 1">加入购物车</div>
              <div class="buy-cart" v-else @click="beforBuy(2)" >加入购物车</div>
              <div class="buybuybuy" @click="beforBuy(1)">立即购买</div>
            </div>
          </div>
        </div>

        <div class="ck-pop-toast" v-if="toast3">
          <div class="inner">
            <div class="ck-pop">
              <div class="ck-pop-close" @click="toastClose(3)"></div>
              <div class="ck-pop-bd">
                <p class="p1">亲，您不是会员，需要原价购买噢！</p>
                <p class="p2">&nbsp;</p>
                <div class="ck-pop-btn" @click="applyGo">立即申请会员</div>
                <p class="p3" @click="buy(2)">原价购买</p>
              </div>
            </div>
          </div>
        </div>
        <!-- 限时够商品数量超过可购买数量提示 -->
        <yd-alert-tip :alertText="tipText" :tipsType='"4"' :closeCallback='toastClose' v-if="toast4"></yd-alert-tip>
      </div>

      <div class="tipsPage" v-if="tips1">
        <div class="tipsBox">
          <div class="tipStatus" data-type="busy"></div>
          <p>亲，网络出问题啦~~</p>
          <div class="btns" @click="reLoad">重新加载</div>
        </div>
      </div>

       <!-- 商品属性弹窗 -->
        <yd-bottom-tip v-if="tips2">
           <section class='format_head'>              
              <div class="pro_info">
                 <div class="pormat_img imgLazy goodsimg" slot="img" v-lazy:background-image="imgBaseUrl+detailsData.picList[0].prodPicId"></div>
                 <div class="pro-mess">
                    <p class="pro-name">{{detailsData.prodTitle}}</p>
                    <!-- <p class="pro-price">￥{{detailsData.memberPrice}}</p> -->
                    <p v-if="detailsData.skipe==true && actionStatu != 2">￥{{detailsData.skipePrice}}</p>
                    <p v-else-if="detailsData.restrict==true && actionStatu != 2">￥{{detailsData.discountPrice}}</p>
                    <p v-else>￥{{detailsData.memberPrice}}</p>
                    <p class="pro-stock" v-if='source == 2 && actionStatu ==1'>库存：{{detailsData.skipeStore}}</p>
                    <p class="pro-stock" v-else>库存：{{detailsData.stocksCount}}</p>
                  </div>                 
              </div>
              <div class='close-tip' @click='tipsCancel'>&nbsp;</div>      
           </section>
           <p class="pro-points" v-if="detailsData.memberPrice-detailsData.pointPrice>0 && detailsData.skipe==false && detailsData.restrict == false">{{detailsData.memberPoint}}积分可兑换{{totalCurPrice}}元现金</p>
           <section class="format_con">           
             <ul class="format-list" v-if="detailsData.guigeList">
                <li v-for="(guigeItem,index) in detailsData.guigeList">
                  <p class="promat_name">{{guigeItem.guigeName}}</p>
                  <div class="item">
                      <span v-for="item in guigeItem.items" class="p_span">
                        <span class="formatNoBtn" v-if="!item.curStatus">{{item.guigeName}}</span>
                        <span v-if="item.curStatus" :class="{'formatCurr':selectFormat[index].guigeId==item.guigeId,'formatNoCurr':selectFormat[index].guigeId!=item.guigeId}" @click='formatSelect(item,index,guigeItem.guigeCode)'>{{item.guigeName}}</span>
                      </span>
                  </div>
                </li>
             </ul>
              <div class="format_count">             
                <span class="details-count-show">购买数量</span>
                <div v-if="stockoun" class="details-stockoun">无货</div>
                <yd-spinner v-if="!stockoun && source==2 && actionStatu ==1" v-model="addProdCount" :class="{'hide1': spinnerWatch(addProdCount,detailsData.skipeStore,'add') == 1 ? 'true' : false, 'hide2': spinnerWatch(addProdCount,detailsData.skipeStore,'minus') == 2 ? true : false}" :max="detailsData.skipeStore"></yd-spinner>
                <yd-spinner v-else v-model="addProdCount" :class="{'hide1': spinnerWatch(addProdCount,detailsData.stocksCount,'add') == 1 ? 'true' : false, 'hide2': spinnerWatch(addProdCount,detailsData.stocksCount,'minus') == 2 ? true : false}" :max="detailsData.stocksCount"></yd-spinner>
              </div>
           </section>
            <div class="ck-fixed-btn disabled" v-if="stockoun">确定</div>
           <div class="ck-fixed-btn" v-if="!stockoun" @click="formatSubmit">确定</div>
        </yd-bottom-tip>

        <!-- 满减送弹窗 -->
        <yd-bottom-tip v-if="tips3" :tipsType='"tips3"'>
          <ul class="action-full-song">
            <li>活动名称：<span>{{appListReduced.title}}</span></li>
            <li class="item1" v-for="reduced in appListReduced.preferentialSets" v-if="reduced.setType==0"><em class="li-em">&nbsp;</em>满<span class="red">{{reduced.threshold}}</span>元，减<span class='red'>{{reduced.reducedMoney}}</span>元。</li>
             <li class="item1" v-for="reduced in appListReduced.preferentialSets" v-if="reduced.setType==1"><em class="li-em">&nbsp;</em>满<span class="red">{{reduced.threshold}}</span>元，送<span class='red'>{{reduced.methods[0].couponAmount}}</span>元优惠券。</li>
          </ul>
          <div class="ck-fixed-btn" @click='tipClose(3)'>完成</div>
        </yd-bottom-tip>

         <!-- 优惠券弹窗 -->
        <yd-bottom-tip v-if="tips4" :isShow='false' :tipsType='"tips4"'>
          <p class="coupon_title">优惠券</p>   
          <ul style="margin-bottom:1.34rem;" class="coupon_list">
            <li class="coupon_item_theme coupon_theme3" v-for='coupon in couponList'>
              <div class='item1'>
                  <div class="title">{{coupon.couponName}}</div>
                  <div class="price">￥<span class='p_1'>{{coupon.couponAmount}}</span></div>
                  <ul class="c_mess">
                      <li><em></em>满{{coupon.leastConsumpMoney}}元可使用</li>
                      <li><em></em>{{coupon.endTime}}到期</li>
                      <li v-if="coupon.prodRestrict==1"><em></em>适用全部商品</li>
                      <li v-else-if="coupon.prodRestrict==2"><em></em>部分商品可用</li>
                  </ul>
              </div>
              <div class="item2" @click='couponBtn(coupon.id)'>
                  <span>立即</span></br>
                  <span>领取</span>
              </div>
            </li>
          </ul>
          <div class="ck-fixed-btn" @click='tipClose(4)'>完成</div>
        </yd-bottom-tip>
    </section>
</template>

<script type="text/babel">
import {setShopsId,getStore,setStore,removeSessionStore,setSessionStore,getSessionStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
export default {
  data() {
    return {
      tips1: false,
      tips2:false,
      tips3:false,
      tips4:false,
      toast1: false, 
      isSelectFormat:false,
      toast3: false,
      toast4: false,
      isSingle: false,
      show: false,
      tipText:'',
      curIndex: '',//记录当前点击的规格下标
      cacehCount:0,
      couponEnable:window.couponEnable,
      totalCurPrice:0,//积分抵扣减免金额
      actionStatu:1,//1、活动开始；2、活动结束；3、活动即将开始
      selectFormat:[{'guigeId':''},{'guigeId':''}],//选择的规格属性ID     
      start1: false,
      stockoun: false,
      source:'',//商品来源：1=限时折扣，2=秒杀；3=优惠券
      title: "商品详情",
      goodsCount: 0,
      addProdCount: 1,
      isCoundownCall:false,//倒计时结束回调重新加载数据
      myCart: {
        prodId: "",
        prodCount: 0
      },
      optAction:{//限时购数据
        currTime:"",
        acStarTime:'',
        acEndTime:'',
        acformat:'{%d}:{%h}:{%m}:{%s}'
      },
      formatType:1,//弹窗打开的来源1、立即购买;2、加入购物车
      imgBaseUrl: window.imgBaseUrl,
      apiUrl: window.baseUrl+window.appName+'/shopapi/func/product/getProductInfo?orgCode='+window.orgCode,
      opts: {"prodId": 0},
      reducedInfo:{},
      detailsData: {},
      //切换商品属性接口
      apiFormatUrl:window.baseUrl+window.appName+'/shopapi/func/product/changeProductGuigeInfo?orgCode='+window.orgCode,
      optFormat:{
        'baseProdId':"",
        'guigeArr':[]//规格属性数组  
      },
      apiCartUrl: window.baseUrl+window.appName+'/shopapi/func/product/addToBasket?orgCode='+window.orgCode,
      optsCart: {},
      apiAddToOrderUrl: window.baseUrl+window.appName+'/shopapi/func/product/addToOrder?orgCode='+window.orgCode,
      //立即预约接口
      apiReserve:window.baseUrl+window.appName+'/shopapi/func/skipe/addReserveRecord?orgCode='+window.orgCode+'&shopsid='+ window.shopsid,
      // 获取满减送列表接口
      apiAppListReduced:window.baseUrl+window.appName+'/shopapi/func/reduced/appListReduced?orgCode='+ window.orgCode +'&shopsid='+ window.shopsid,
      //领取优惠券
      apiAddUserCoupon:window.baseUrl+window.appName+'/shopapi/func/coupon/addUserCoupon?orgCode='+ window.orgCode +'&shopsid='+ window.shopsid,
      optsAddToOrder: {
        prodIds: [],
        genOrderSource: 2,
        isContinue: 2
      },
      total: 0,
      cartDetailsData: [],
      couponList:[],
      cartStatus:false,//加入购物车按钮状态
      appListReduced: {}//满减送
    }
  },
  watch: {
    goodsCount() {  //深度监商品数据,用于显示购物车ICON红色微标
      if(this.goodsCount > 0 ) {
        this.show = true;
      }else {
        this.show = false;
      }
    },
    '$route.query.pid': 'markInit' //监听当前路由变化（辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
    this.fetchDetailsData(this.log);
  },
  methods: {
    reLoad() {
      //重载页面
      this.$router.go(0);
      this.markInit();
    },
    log(mod) {
      var sItem = {};
          sItem.itemid = this.myCart.prodId, 
          sItem.name = this.detailsData.prodTitle, 
          sItem.price = this.detailsData.memberPrice, 
          sItem.picid = this.detailsData.onePicId;
      mod = mod || LOGCODE.HOME.PAGE_DETAIL;
      this.$store.dispatch('log', {account:mod, ShopItem: sItem});
    },
    toastClose(toast) {
      //关闭提示框
      this['toast'+toast] = false;
    },
    tipClose(tips){
      this['tips'+tips] = false;
      // document.getElementsByTagName('body')[0].removeAttribute('style','overflow:hidden');
    },
    openTip(tips){
      this['tips'+tips] = true;
      // document.getElementsByTagName('body')[0].style='overflow:hidden';
    },
    applyGo() {
      //跳去申请会员
      this.$router.push('/register');
    },
    cartGo() {
      //跳转到购物车
      this.$router.push('/cart');
      // sessionStorage.removeItem('goods_nav_menu');
      removeSessionStore('goods_nav_menu');
    },
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取商品ID
      this.myCart.prodId = this.$route.query.pid;
      //获取商量列表进详情缓存基本信息
      this.source = this.$route.query.psource ? this.$route.query.psource :'1';
      // console.log({'source':this.source});
      //获取购物车缓存数据
      let _saveCart = getStore('_saveCart',true);
      if(typeof(_saveCart) ==='boolean'){
        this.goodsCount = 0;
      }else{
        this.goodsCount = _saveCart.length;
      }
      //获取shopsid缓存数据
      setShopsId();

    },
    spinnerWatch(count,stocks,flag) {
      //【+】【-】控件数据监听方法
      if(flag == 'add') {
        if(count == stocks) {
          return 1;   
        }
      }else {
        if(count == 1 || count == 0) {
          return 2;
        }
      }
    },
    addCart() {
      //加入购物车并保存数据
      const that = this;
      // let _saveCart = JSON.parse(localStorage.getItem("_saveCart"));
      let _saveCart = getStore('_saveCart',true);
      let _newCart = [];
      let _hasrepeat = false;
      // if(_saveCart == null || _saveCart == 'undefined') {
      if(typeof(_saveCart) === 'boolean') {
        _saveCart = [];
      }
      if(_saveCart.length != 0) {
        _saveCart.forEach(function(item,i) {
            if(item.prodId == that.myCart.prodId) {
              that.myCart.prodCount = item.prodCount + that.addProdCount;
              if(that.myCart.prodCount > that.detailsData.stocksCount) {
                that.myCart.prodCount = that.detailsData.stocksCount;
                that.$dialog.toast({mes: '此商品库存不足噢~', timeout: 1200});
              }
              item.prodCount = that.myCart.prodCount;
              _hasrepeat = true;
            }
            _newCart.push(item);
        });
        if(_hasrepeat == false) {
          // console.log(that.myCart.prodCount+'&&'+that.addProdCount);
          // that.myCart.prodCount = that.myCart.prodCount + that.addProdCount;
          that.myCart.prodCount = that.addProdCount;
          if(that.myCart.prodCount > that.detailsData.stocksCount) {
            that.myCart.prodCount = that.detailsData.stocksCount;
            that.$dialog.toast({mes: '此商品库存不足噢~', timeout: 1200});
          }
          _newCart.push(that.myCart);
        }
      }else {
        that.myCart.prodCount = that.myCart.prodCount + that.addProdCount;
        if(that.myCart.prodCount > that.detailsData.stocksCount) {
          that.myCart.prodCount = that.detailsData.stocksCount;
          that.$dialog.toast({mes: '此商品库存不足噢~',timeout: 1200});
        }
        _newCart.push(that.myCart);
      }
      _saveCart = _newCart;
      //提交购物车数据
      console.log(_saveCart);
      that.optsCart.prodIds = _saveCart;
      that.saveCartList();
      that.goodsCount = _saveCart.length;
      that.$store.commit('UPDATE_GoodsCount', that.goodsCount);
      that.$dialog.notify({mes: '购物车已有 '+that.myCart.prodCount+' 件此商品', icon: 'success', timeout: 1200});
      //缓存购物车数据
      setStore("_saveCart",_saveCart);
      setStore("_saveCartCount",that.goodsCount);
      // sessionStorage.setItem("_lastProdId", that.myCart.prodId); //最新添加商品ID缓存
      setSessionStore("_lastProdId", that.myCart.prodId);
      this.log(LOGCODE.HOME.PAGE_DETAIL_BTN_CART_ADD);//加入购物车
    },
    //设置积分抵扣价是否有小数点
    setPointPrice(){
      let _totalCurPrice = this.detailsData.memberPrice-this.detailsData.pointPrice;
      if(parseInt(_totalCurPrice.toFixed(2).split('.')[1])>0){
        this.totalCurPrice = _totalCurPrice.toFixed(2);
      }else{
        this.totalCurPrice = _totalCurPrice;
      }
    },
    //从详情页底菜单点立即购买触发此按钮,选择商品属性以及确定商品数量
    beforBuy(pIndex){
        this.tips2 = true;
        this.formatType = pIndex;
        if(this.detailsData.restrictType==2 && pIndex==1){
          this.cacehCount = this.detailsData.stocksCount;
          this.detailsData.stocksCount = this.detailsData.restrictCount;
        }     
    },
    formatSubmit(){
      if(this.detailsData.restrictType==2&&this.detailsData.restrictCount<this.addProdCount&&this.formatType==1){
        this.toast4 = true;
        this.tipText = '亲，此限购商品每人只能购买<span class="red">'+this.detailsData.restrictCount+'</span>件';
        return;
      }
      if(this.detailsData.guigeList.length>0){
          if(this.isSelectFormat){
            this.tips2 = false;
            if(this.formatType==1){
              this.buy(2);
            }else{
              this.addCart();
            }        
          }else{
            this.$dialog.notify({mes: '请选择规格属性', icon: 'error', timeout: 1200});
          }
      }else{
        this.tips2 = false;
        if(this.formatType==1){
          this.buy(2 );
        }else{
          this.addCart();
        }   
      }     
    },
    buy(isContinue) {
      this.log(LOGCODE.HOME.PAGE_DETAIL_BTN_BUY);

      //点击购买按钮触发相关数据请求逻辑
      let that = this;
      // if(that.myCart.prodCount == 0) {
      //   that.myCart.prodCount = that.addProdCount;
      // }
      if(that.source==2 || that.source ==3){//限时进来的商品详情
        that.optsAddToOrder.activityId = that.$route.query.activityId;
      }
      that.myCart.prodCount = that.addProdCount;
      that.optsAddToOrder.prodIds[0] = that.myCart;
      that.$dialog.loading.open('请求数据中...');
      that.$http.post(that.apiAddToOrderUrl+'&shopsid='+window.shopsid + '&fromSource='+this.source,that.optsAddToOrder)
        .then((response) => {
          const data = response.data;
          that.$dialog.loading.close();

          if(data.code == "S_OK") {
            //跳去订单详情
            // sessionStorage.removeItem('goods_nav_menu');
            removeSessionStore('goods_nav_menu');
            that.$router.push({
                path: '/orderdetails',
                query: {
                  source: 2,
                  oid: data.var
                }
            });
          }else{
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else if(data.code == "USER_NOT_MEMBER"){
              that.toast3 = true;
            }else if(data.code == "PRODUCT_BUY_COUNT_EXCEED"){
              that.toast4 = true;
              that.tipText = '亲，此限购商品每人只能购买<span class="red">'+that.detailsData.restrictCount+'</span>件';
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          that.$dialog.loading.close();
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
        });
    },
    fetchDetailsData(cb) {
      //异步请求商品详情数据
      let that = this;
      that.opts.prodId = this.$route.query.pid;
      that.$dialog.loading.open('加载数据中...');
      if(this.$route.query.psource==4){//优惠券进来的商品详情
        // that.opts.couponId = sessionStorage.getItem('couponId');
        that.opts.couponId = getSessionStore('couponId',false);
      }
      if(this.$route.query.psource==2 || this.$route.query.psource==3){//优惠券进来的商品详情
        that.opts.activityId = this.$route.query.activityId;
      }
      that.$http.post(that.apiUrl+'&fromSource='+this.source,that.opts) 
      // that.$http.post(that.apiUrl+'&fromSource=1',that.opts)
        .then((response) => {
          const data = response.data;
          that.tips1 = false;
          if(data.code == "S_OK") {
            that.detailsData = data.var.productInfo;
            that.couponList = data.var.applicableCouponList;
            // console.log(data.var.productInfo);
            that.reducedInfo = data.var.reducedInfo;
             //积分抵扣减免金额是否有保留两位小数
             that.setPointPrice();
            //限时购判断时间
              that.optAction.currTime = that.detailsData.currentTime;
              let _currTime = Math.floor(new Date((that.optAction.currTime).replace(/-/g,"\/")).getTime() / 1000);
              let actionEndTime = Math.floor(new Date((that.detailsData.endTime).replace(/-/g,"\/")).getTime() / 1000);
              if(_currTime < actionEndTime || _currTime == actionEndTime){
                  if(that.detailsData.isCyclePepeat ==1){//有重复周期                      
                    let _currDate = that.detailsData.currentTime.split(' ')[0];
                    let curDate = new Date((that.optAction.currTime).replace(/-/g,"\/"));
                    let endDate = new Date((that.detailsData.endTime).replace(/-/g,"\/"));
                    let cycleStartMinute = that.detailsData.cycleStartMinute>9?that.detailsData.cycleStartMinute:'0'+that.detailsData.cycleStartMinute;
                    let newEndDate = Math.floor(endDate.getTime() / 1000)-_currTime;
                    that.optAction.acStarTime = _currDate +' '+ that.detailsData.cycleStartHour +':'+ cycleStartMinute+':00';
                    that.optAction.acEndTime = _currDate +' '+ that.detailsData.cycleEndHour +':'+ that.detailsData.cycleEndMinute+':00';
                    //判断今天抢购活动是否结束
                      if(Math.floor(curDate.getTime() / 1000)>Math.floor(new Date((that.optAction.acEndTime).replace(/-/g,"\/")).getTime() / 1000)){             
                        if(newEndDate>0){
                          that.optAction.acStarTime = curDate.getFullYear()+'-'+(curDate.getMonth()+1)+'-'+(curDate.getDate()+1) +' '+ that.detailsData.cycleStartHour +':'+ that.detailsData.cycleStartMinute+':00';    
                        }
                      }
                  }else{//无重复周期
                    that.optAction.acStarTime = that.detailsData.startTime;
                    that.optAction.acEndTime = that.detailsData.endTime;
                  }
                  var _starTime = Math.floor(new Date((that.optAction.acStarTime).replace(/-/g,"\/")).getTime() / 1000);
                  var _endTime = Math.floor(new Date((that.optAction.acEndTime).replace(/-/g,"\/")).getTime() / 1000);
                  console.log('2=>',_starTime,_endTime,that.optAction.acStarTime,that.optAction.acEndTime,that.isCoundownCall);
                  if(_currTime<_starTime){//活动未开始
                    that.actionStatu = 3;      
                  }else if((_starTime==_currTime || _starTime <_currTime)&&_currTime<_endTime){//活动开始
                      that.actionStatu = 1;    
                    if(that.isCoundownCall){
                      console.log('count=>',that.optAction);
                      that.$refs.countDown.run(that.optAction);
                    } 
                  }else{//活动已结束
                    that.actionStatu = 2; 
                  }
              }else{
                that.actionStatu = 2;
              }
              console.log(that.optAction,that.detailsData);
            //判断是否可以加入购物车
            if((that.detailsData.register || that.detailsData.skipe) && that.detailsData.status != 3){
              that.cartStatus = true;
            }else if ((that.detailsData.register || that.detailsData.skipe) && that.actionStatu == 3){
              that.cartStatus = false;
            }
            if(that.detailsData.stocksCount == 0) {
              that.stockoun = true;
            }
            if(that.detailsData.picList.length == 1) {
              that.isSingle = true;
            }else {
              that.start1 = true;
            }
            //初始化格属性是否可点击状态，供活动商品选择属性时设置没有参与活动的属性为不可点击状态
            if(that.detailsData.guigeList.length>0){
              if(that.detailsData.guigeList.length==1){//单个规格直接初始化
                var guiItem = that.detailsData.guigeList[0].items;
                for(var i=0;i<guiItem.length;i++){
                  if(that.source == 3){
                    var eachItem = that.detailsData.exists.filter(function(data){//从可选择属性数组中筛选匹配项
                      return data.guigeVal1 == guiItem[i].guigeCode;
                    });
                    that.detailsData.guigeList[0].items[i].curStatus = eachItem.length>0? true:false;
                  }else{
                    that.detailsData.guigeList[0].items[i].curStatus = true;
                  }
                }
              }else{//多规格初始化为可点击
                for(var i=0;i<that.detailsData.guigeList.length;i++){
                  var item = that.detailsData.guigeList[i];
                  for(var j=0;j<item.items.length;j++){
                    var itemJ = item.items[j];
                    that.detailsData.guigeList[i].items[j].curStatus = true;
                  }
                }
              }  
            }
          }else{
            that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
          }
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);

          if (typeof cb == 'function') {
            cb();
          }

        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    saveCartCache() {
      //缓存购物车列表
      let that = this;
      let _cartDetailsData = getStore("_cartDetailsData",true);
      let _newCart = [];
      let _hasrepeat = false;
      that.detailsData.prodCount = that.myCart.prodCount;
      if(_cartDetailsData === 'boolean') {
        _cartDetailsData = [];
      }
      if(_cartDetailsData.length != 0) {
        _cartDetailsData.forEach(function(item,i) {
          if(item.id == that.myCart.prodId) {
            item.prodCount = that.myCart.prodCount;
            _hasrepeat = true;
          }
          _newCart.push(item);
        });
        if(_hasrepeat == false) {
          that.detailsData.id = that.myCart.prodId;
          _newCart.push(that.detailsData);
        }
      }else {
        that.detailsData.id = that.myCart.prodId;
        _newCart.push(that.detailsData);
      }
      _cartDetailsData = _newCart;
      //缓存详情数据
      setStore("_cartDetailsData", _cartDetailsData);
    },
    saveCartList() {
      //保存购物车列表（异步提交/本地缓存）
      let that = this;
      that.$http.post(that.apiCartUrl+'&shopsid='+window.shopsid,that.optsCart)
        .then((response) => {
          const data = response.data;
          if(data.code == "S_OK") {

          }else if(data.code == "NO_LOGIN" || data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
            //缓存购物车列表
            that.saveCartCache();
          }else{
            that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
          }
        });
    },
    //获取规格属性数据，实时更新数据
    fetchDetailsFormatData(){
      let that = this;
      var _url = that.apiFormatUrl;
        that.optFormat.baseProdId = that.detailsData.baseProdId;
        if(this.source ==2 || this.source == 3){
          that.optFormat.activityId = this.$route.query.activityId;
          if(this.source == 3){
            _url = _url +'&fromSource='+this.source;
          }
        }

        that.$http.post(_url,that.optFormat)
        .then((response) => {
          const data = response.data;
          that.tips1 = false;
          if(data.code == "S_OK") {
            that.detailsData.stocksCount = data.var.stocksCount;
            that.detailsData.prodId = data.var.id;
            that.myCart.prodId = data.var.id;
            if(this.source==3){
              that.detailsData.discountPrice = data.var.discountPrice;
            }else{
              that.detailsData.memberPrice = data.var.memberPrice;
              that.detailsData.orgPrice = data.var.orgPrice;
              that.detailsData.memberPoint = data.var.memberPoint;
              that.detailsData.pointPrice = data.var.pointPrice;
            }
            that.setPointPrice();
            if(that.detailsData.stocksCount == 0) {
              that.stockoun = true;
            }else{
              that.stockoun = false;
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
    //判断此规格属性是否可选
    setGuiFormat(index,guigeCode,pBol){
      var returnBol = false;
      var _index = index==0? 1:0;//点的是规格1，筛选的就是规格2里的属性
      var n = index == 0? 2:1;//可点规格属性筛选也是同样的原理
      var arr = this.detailsData.exists.filter(function(data){//可点击属性数组里找到当前点击属性的组合
            return data['guigeVal'+(1+parseInt(index))] == guigeCode;
          });
      console.log({'index':index,'arr':arr});
      for(var i=0;i<this.detailsData.guigeList[_index].items.length;i++){
        var item = this.detailsData.guigeList[_index].items[i];
        if(pBol){       
          var c_len = arr.filter(function(data){
            return item.guigeCode == data['guigeVal'+n];
          });
          if(c_len.length>0){
            this.detailsData.guigeList[_index].items[i].curStatus = true;
          }else{
            this.detailsData.guigeList[_index].items[i].curStatus = false;
            if((typeof this.optFormat.guigeArr[_index])=="object" && this.optFormat.guigeArr[_index].guigeValue==this.detailsData.guigeList[_index].items[i].guigeCode){
                this.optFormat.guigeArr[_index] = {};
                this.selectFormat[_index].guigeId = '';
                this.optFormat.guigeArr[_index].guigeCode = '';
                this.optFormat.guigeArr[_index].guigeValue = '';
              }
          }
        }else{
          this.detailsData.guigeList[_index].items[i].curStatus = true;
        }       
      }
    },
    //规格属性行选择
    formatSelect(pThis,pIndex,pGuigeCode){
      if(this.selectFormat[pIndex].guigeId == pThis.guigeId){ //点击的是已经勾选的项
        this.selectFormat[pIndex].guigeId = '';
        this.optFormat.guigeArr[pIndex].guigeCode = '';
        this.optFormat.guigeArr[pIndex].guigeValue = '';
        this.isSelectFormat = false;
        if(this.source==3 && this.detailsData.guigeList.length >1){
          this.setGuiFormat(pIndex,pThis.guigeCode,false);
        }
      }else{
        this.optFormat.guigeArr[pIndex] = {};
        this.optFormat.guigeArr[pIndex].guigeCode = pGuigeCode;
        this.optFormat.guigeArr[pIndex].guigeValue = pThis.guigeCode;

        if(this.source==3 && this.detailsData.guigeList.length >1){
          this.setGuiFormat(pIndex,pThis.guigeCode,true);
        }
     
        let len = this.optFormat.guigeArr.filter(function(data){
          return data !=null && data !=undefined && data.guigeCode !=''&& data.guigeValue !='';
        });
        //当规格不为空时，判断是否所有规格都勾选了属性
        if(this.detailsData.guigeList.length>0){
          if(len.length==this.detailsData.guigeList.length){
            this.isSelectFormat = true;
            this.fetchDetailsFormatData(); 
          }else{
            this.isSelectFormat = false;
          } 
          this.selectFormat[pIndex].guigeId = pThis.guigeId;      
        }else{
          this.isSelectFormat = true;
        }     
      }  
    },
    //活动倒计时结束回调
    countDownCall(){
      console.log('countDownBack');
      this.isCoundownCall = true;
      this.fetchDetailsData(this.log);
    },
    //秒杀活动立即预约
    reserveBtn(str) {
      let that = this;
        this.log(LOGCODE.HOME.PAGE_DETAIL_BTN_RESERVE);
        that.$http.post(that.apiReserve,{"activityId":str,'prodId':this.myCart.prodId})
        .then((response) => {
          let data = response.data;
          if(data.code == "S_OK") {
            that.$dialog.notify({mes: '预约成功', icon: 'error', timeout: 2000});
            that.log(LOGCODE.HOME.PAGE_DETAIL_MESS_RESERVE_OK);
          }else{
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else{
              that.log(LOGCODE.HOME.PAGE_DETAIL_MESS_RESERVE_FAIL);
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    //领取优惠券
    couponBtn(str){
      let that = this;
      this.log(LOGCODE.HOME.PAGE_DETAIL_BTN_COUPON_ADD);
        that.$http.post(that.apiAddUserCoupon,{'couponId':str,'couponCount':1})
        .then((response) => {
          const data = response.data;
          if(data.code == "S_OK") {
            that.$dialog.notify({mes: '领取成功', icon: 'error', timeout: 2000});
            that.log(LOGCODE.HOME.PAGE_DETAIL_MESS_COUPON_ADD_OK);
          }else{
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    //获取满减送列表
    getAppListReduced(tips) {
       let that = this;
        that.$http.post(that.apiAppListReduced)
        .then((response) => {
          const data = response.data;
          if(data.code == "S_OK") {
            that.appListReduced = data['var'];
            that.openTip(tips);
            // that['tips'+tips] = true;
          }else{
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    tipsCancel(){
      this.tips2 = false;
      // this.detailsData.stocksCount = this.cacehCount;
    }
  }
}
</script>
