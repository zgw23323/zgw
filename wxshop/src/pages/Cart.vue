<template>
<!-- <yd-layout> -->
<div>
  <section class="cart">
  <div class="cart_song_mess" v-if="!tips1 && appListReduced.length>0">
         <div class="item">
            <span class="c_title">满减</span>
            <span class="c_mess">{{reducedMess}}</span>
         </div>
      </div>
    <!-- <yd-infinitescroll :on-infinite="loadList" v-if="!tips2"> -->
    <div v-if="!tips2">
     <!-- 满减送信息 -->
      
      <yd-list slot="list">
        <ul :class="{'cart-goods m-top': appListReduced.length>0,'cart-goods':appListReduced.length == 0}">
            <!-- <yd-leftSlider :index="index" key='index' v-for="(cart,index) in cartList" :class="{'del': showDelvalidator(cart.id) ? true : false}"> -->
             <li v-for="(cart,index) in cartList">
              <div class="edit_btn" @click='editDeleteBtn(cart.id)' v-if="editId != cart.id">编辑</div>
              <div class="edit_btn" @click='editDeleteBtn(cart.id)' v-if="editId == cart.id">完成</div>
              <div class="cl cart_li_cl">
                <div class="item">
                  <div class="cart-goods-checkbox" @click="curSelected(cart.id)">
                    <div class="cart-items-checkbox" :class="{'selected': selectedValidator(cart.id) ? true : false}"></div>
                  </div>
                  <div class="cart-goods-img imgLazy" v-lazy:background-image="imgBaseUrl+cart.prodPic" @click="detailsGo(cart.id)"></div>
                </div>
                <div class="cart-goods-mes" v-if="editId != cart.id">
                  <div class="cart-goods-title"><span>{{cart.prodTitle}}</span></div>
                  <p class="cart-goods-format">{{cart.setProdGuigeDesc}}</p>                  
                 <div class="car-price-count">
                    <p class="od-goods-memberPrice">￥{{cart.memberPrice}}</p>
                    <div class="cart-item-count">x {{cart.prodCount}}</div>
                  </div>
                  <!-- <div class="action_logo" v-if="cart.restrict==true"></div> -->
                </div>
                <!-- 编辑购物车 -->
                <div class="edit_cart" v-if="editId == cart.id">
                  <div :class="{'edit_mess': cart.setProdGuigeDesc=='','edit_mess1':cart.setProdGuigeDesc !=''}">
                    <div class="cart-goods-count" v-if="cart.restrict&&cart.restrictType==2">
                         <yd-spinner v-if="useCache" v-model="cart.prodCount" :class="{'hide1': spinnerWatch(cart.id,cart.prodCount,cart.restrictCount,'add',index) == 1 ? true : false, 'hide2': spinnerWatch(cart.id,cart.prodCount,cart.restrictCount,'minus',index) == 2 ? true : false}" :min="0" :max="cart.restrictCount"></yd-spinner>
                        <yd-spinner v-if="!useCache" v-model="cart.prodCount" :class="{'hide1': spinnerWatch(cart.baskId,cart.prodCount,cart.restrictCount,'add',index) == 1 ? true : false, 'hide2': spinnerWatch(cart.baskId,cart.prodCount,cart.restrictCount,'minus',index) == 2 ? true : false}" :min="0" :max="cart.restrictCount"></yd-spinner>
                      </div>
                      <div class="cart-goods-count" v-else>
                        <yd-spinner v-if="useCache" v-model="cart.prodCount" :class="{'hide1': spinnerWatch(cart.id,cart.prodCount,cart.stocksCount,'add',index) == 1 ? true : false, 'hide2': spinnerWatch(cart.id,cart.prodCount,cart.stocksCount,'minus',index) == 2 ? true : false}" :min="0" :max="cart.stocksCount"></yd-spinner>
                        <yd-spinner v-if="!useCache" v-model="cart.prodCount" :class="{'hide1': spinnerWatch(cart.baskId,cart.prodCount,cart.stocksCount,'add',index) == 1 ? true : false, 'hide2': spinnerWatch(cart.baskId,cart.prodCount,cart.stocksCount,'minus',index) == 2 ? true : false}" :min="0" :max="cart.stocksCount"></yd-spinner>                      
                      </div>
                      <p class="cart-edit-format">{{cart.setProdGuigeDesc}}</p>
                    </div>
                  <div class="cart-del" @click="toastOpen(4,cart.id)" v-if="useCache">删除</div>
                  <div class="cart-del" @click="toastOpen(4,cart.baskId)" v-if="!useCache">删除</div> 
                </div>
              </div>
              <!-- <div> -->
              <!-- <div class='edit_delete_bg'></div> -->
              <!-- <div class="cart-goods-del" @click="deleteBasket(cart.id,index)" v-if="useCache">删除</div> -->
              <!-- <div class="cart-goods-del" @click="deleteBasket(cart.baskId,index)" v-if="!useCache">删除</div>  -->
              </li>
            <!-- </yd-leftSlider> -->
        </ul>
      </yd-list>
    </div>
    <!-- </yd-infinitescroll> -->
    <div class="cart-ft-price-bar" v-if="show">
      <div class="inner" v-if="!isVip">
        <div class="cart-ft-price-btn" @click="selectedAll">
          <div class="cart-ft-checkbox" :class="{'selected': isSelectedAll ? true : false}"></div>
        </div>
        <div class="cart-ft-price-btn" @click="buy(2)">
          <p>￥{{orgPriceCount}}</p>
          <p>原价购买</p>
        </div>
        <div class="cart-ft-price-btn" @click="buy(2)">
          <p>￥{{orgPointPriceCount}}</p>
          <p>会员价购买</p>
        </div>
      </div>
      <div class="inner flexStyle" v-if="isVip">
        <div class="cart-ft-price-btn" @click="selectedAll">
          <div class="cart-ft-checkbox" :class="{'selected': isSelectedAll ? true : false}"></div>
        </div>
        <div class="cart-ft-price-btn2-1">
          <p class="p1">合计:</p>
          <p class="p2">￥{{orgPointPriceCount}}+{{orgPointCount}}<em>积分</em></p>
        </div>
        <div class="cart-ft-price-btn2-2" @click="buy(2)">立即支付</div>
      </div>
    </div>

    <ck-nav></ck-nav>

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
       
    <div class="tipsPage" v-if="tips1">
      <div class="tipsBox">
        <div class="tipStatus" data-type="cart"></div>
        <p>亲，购物车内空空如也~~</p>
        <router-link to="/goods" class="btns">去逛逛</router-link>
      </div>
    </div>

    <div class="tipsPage" v-if="tips2">
      <div class="tipsBox">
        <div class="tipStatus" data-type="busy"></div>
        <p>亲，网络出问题啦~~</p>
        <div class="btns" @click="reLoad">重新加载</div>
      </div>
    </div>
    
    <!-- 限时够商品数量超过可购买数量提示 -->
    <yd-alert-tip :alertText="tipText" :tipsType='"4"' :isCallback='true' :closeCallback='toastClose' :callback='deleteCallback' v-if="toast4"></yd-alert-tip>
  </section>
</div>
<!-- </yd-layout> -->
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,getStore,setStore,getSessionStore,removeSessionStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
export default { 
    data() {
      return {
          title: '购物车',
          useCache: false,
          isVip: false,
          tips1: false,
          toast3: false,
          toast4:false,
          tips2: false,
          show: false,
          tipText:'确定要删除该商品吗？',
          showEdit:'编辑',
          editId:'',
          isLoading:true,
          isDone:false,
          deleteCartId:'',
          isSelectedAll: false,
          imgBaseUrl: window.imgBaseUrl,
          apiCartUrl: window.baseUrl+window.appName+'/shopapi/func/product/queryBasketList?orgCode='+window.orgCode,
          optsCart: {
            curPage: 1,
            pageSize: 7
          },
          cartList: [],
          cartListLen: 0,
          curSelectedId: [],
          curShowDelId: [],
          orgPriceCount: 0,
          orgPointPriceCount: 0,
          orgPointCount: 0,
          restrictCount:0,//限购数量
          apiDeleteBasketUrl: window.baseUrl+window.appName+'/shopapi/func/product/deleteBasket?orgCode='+window.orgCode,
          optsDeleteBasket: {
            baskIds: []
          },
          apiAddToBasketUrl: window.baseUrl+window.appName+'/shopapi/func/product/addToBasket?orgCode='+window.orgCode,
          optsAddToBasket: {},
          saveCart: {},
          apiAddToOrderUrl: window.baseUrl+window.appName+'/shopapi/func/product/addToOrder?orgCode='+window.orgCode,
          optsAddToOrder: {
            prodIds: [],
            genOrderSource: 1,
            isContinue: 2
          },
          // 获取满减送列表接口
          apiAppListReduced:window.baseUrl+window.appName+'/shopapi/func/reduced/appListReduced?orgCode='+ window.orgCode +'&shopsid='+ window.shopsid,
          appListReduced: [],//满减送
          reducedMess:'',//满减送提示
          isReducedFull:false,//是否满足满减送等级
          reducedTotalPrice:0,
          cartSave: true
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchCartList();
      // this.run(1);
      //监听滚动条事件
      window.addEventListener('scroll', this.menu);
    },
    watch: {
      cartList: {  //深度监听购物车列表数据
        handler() {
          //计算总价格
          this.priceTotal(this.curSelectedId);
        },
        deep: true
      },
      curSelectedId: {  //深度监听当前选择数据
        handler() {
          //计算总价格
          this.priceTotal(this.curSelectedId);
        },
        deep: true
      },
      '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    computed: {
      ...mapState([
        'goodsCount'
      ])
    },
    methods: {
      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
          if(this.optsCart.total > this.optsCart.curPage){
            this.loadList();
            // this.isDone = false;
            this.isLoading = true;
          }else{
            // this.isDone = true;
            this.isLoading = false;
          }
        }
      },
      //定时请求活动数据
      run(n) {
        this.timer = setInterval(() => {
            if(n==1){
              this.getAppListReduced();
            }else if(n == 2){
              // console.log(this);
              clearInterval(this.timer);
            }  
        }, 10000);
      },
      detailsGo(id) {
        //跳转到详情
        this.$router.push({
          path: '/details',
          query: {
            pid: id
          }
        });
      },
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
      spinnerWatch(id,count,stocks,flag,index) {
        //【+】【-】控件数据监听方法
        if(flag == 'add') {
          if(count == stocks) {
            return 1;
          }
        }else {
          if(count == 0) {
            this.deleteBasket(id,index);
            return 2;
          }else if(count == 1) {
            return 2;
          }
        }
      },
      editDeleteBtn(id){
        if(this.editId == id){
          this.editId = '';
          this.showEdit = false;
        }else{
          this.showEdit = true;
          this.editId = id;
        } 
      },
      applyGo() {
        //跳去申请会员
        this.$router.push('/register');
      },
      selectedAll() {
        //选择购物车全部
        this.isSelectedAll = !this.isSelectedAll;
        this.curSelectedId = [];
        if(this.isSelectedAll) {
          for(let i = 0; i < this.cartList.length; i++) {
            this.curSelectedId.push(this.cartList[i].id);
          }
        }else {
          this.curSelectedId = [];
        }
      },
      loadList() {
        //分页加载异步请求数据
        let that = this;
        that.optsCart.curPage++;
        if(that.optsCart.curPage > that.optsCart.total) {
          /* 所有数据加载完毕 */
          that.optsCart.curPage = that.optsCart.total;
          that.isLoading = false;
          // that.isDone = true;
          // that.$yduiBus.$emit('ydui.infinitescroll.loadedDone');
          return;
        }
        that.$http.post(that.apiCartUrl+'&shopsid='+window.shopsid,that.optsCart)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
               //that.cartList = [...that.cartList, ...data.var];
                for (let list of data.var.baskList) {
                  that.cartList.push(list);
                }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }

            /* 单次请求数据完毕 */
            // that.$yduiBus.$emit('ydui.infinitescroll.finishLoad');

            // that.optsCart.curPage++;
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      toastOpen(toast,id) {
        //关闭弹出提示框
        this.deleteCartId = id;
        this['toast'+toast] = true;
      },
      toastClose(toast) {
        //关闭弹出提示框
        this['toast'+toast] = false;
      },
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        //获取最近添加商品缓存
        var _lastProdId = getSessionStore("_lastProdId",false);
        if(typeof('_lastProdId')==='boolean') {
          this.curSelectedId = [];
        }else {
          this.curSelectedId.push(_lastProdId);
        }
        //获取购物车缓存数据
        var _saveCart = getStore('_saveCart',true);
        if(typeof('_saveCart')==='boolean'){
          this.saveCart = {}
        }else{
          this.saveCart = _saveCart;
        }
        //是否会员
        if(window.isVip === true) {
          this.isVip = true;
        }else {
          this.isVip = false;
        }

        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_CART});
      },
      deleteCallback(){
        // console.log('callback');
        this.deleteBasket(this.deleteCartId);
        this.toast4 = false;
      },
      deleteBasket (baskId) {
        //移除购物车列表
        let that = this;
        that.optsDeleteBasket.baskIds[0] = { 'baskId': baskId };
        this.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_CART_DELETE});
        that.$http.post(that.apiDeleteBasketUrl+'&shopsid='+window.shopsid,that.optsDeleteBasket)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              for(var i = 0; i < that.cartList.length; i++){
                if(baskId == that.cartList[i]['baskId']) {
                  that.cartList.splice(i,1);
                  if(that.cartList.length ==0){
                    that.optsCart.curPage=1;
                    that.fetchCartList();
                  }
                  break;
                }
              }
              that.$store.commit('UPDATE_GoodsCount', that.goodsCount-1);
              // localStorage.setItem('_saveCartCount',that.goodsCount);
              setStore('_saveCartCount',that.goodsCount-1);

              this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_CART_DELETE_OK});
            }else{
              if(data.code == "NO_LOGIN" || data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                console.log('deleteGood-》',that.cartList);
                for(let i = 0; i < that.cartList.length; i++){
                  if(baskId == that.cartList[i]['id']) {
                    that.cartList.splice(i,1);
                    // document.getElementsByTagName("li")[index].style = "transform:translateX(0)";
                    this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_CART_DELETE_OK});
                    break;
                 }
                }
                if(that.cartList.length == 0 ) {
                  that.show = false;
                  that.tips1 = true;
                }else {
                  that.show = true;
                  that.priceTotal(that.curSelectedId);
                }
                console.log('goodsCount=>',that.goodsCount);
                that.$store.commit('UPDATE_GoodsCount', that.goodsCount-1);
                setStore('_saveCartCount',that.goodsCount-1);
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      curShowDel(id) {
        //显示删除按钮
        if(!this.showDelvalidator(id)) {
          this.noRepeatAddArray(id, this.curShowDelId);
        }
      },
      curHideDel(id) {
        //隐藏删除按钮
        if(this.showDelvalidator(id)) {
          this.noRepeatAddArray(id, this.curShowDelId);
        }
      },
      curTapDel(id) {
        //获取当前删除购物车列表的id数组
        this.noRepeatAddArray(id, this.curShowDelId);
      },
      showDelvalidator(value) {
        //是否显示删除按钮id校验
        return this.curShowDelId.indexOf(value) > -1;
      },
      selectedValidator(value) {
        //是否显示选择按钮id校验
        return this.curSelectedId.indexOf(value) > -1;
      },
      curSelected(id) {
        //获取当前选择购物车列表的id数组
        this.noRepeatAddArray(id, this.curSelectedId);
        //当列表全选时全选按钮高亮
        if(this.curSelectedId.length == this.cartList.length) {
          this.isSelectedAll = true;
        }else {
          this.isSelectedAll = false;
        }
      },
      noRepeatAddArray(selected, selectArr) {
        //数组去重过滤方法
        let len = selectArr.length + 1;
        for(let i = 0; i < len; i++){
            if(selected == selectArr[i]) {
              selectArr.splice(i,1);
              return;
            }
        };
        selectArr.push(selected);
      },
      setAttrLi(){
        var domLi = document.getElementsByTagName("li");
        for(var i=0;i<domLi.length;i++){
          domLi[i].style="transform:translateX(0rem)";
        }
      },
      fetchCartList() {
        //异步请求获取购物车列表
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiCartUrl+'&shopsid='+window.shopsid,this.optsCart)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            that.tips2 = false;
            if(data.code == "S_OK") {
              that.cartList = data.var.baskList;
              that.optsCart.total = data.var.totalPage;
              //判断当前页与总页数
              if(data.var.totalPage){//购物车为空
                if(that.optsCart.curPage==data.var.totalPage){
                  // that.isDone=true;
                  that.isLoading = false;
                }
              }else{
                // that.isDone=false;
                that.isLoading = false;
              }
              
              if(that.cartList.length == 0 &&that.optsCart.total==0) {
                that.show = false;
                that.tips1 = true;
              }else {
                that.tips1 = false;
                that.show = true;
                //获取满减送列表
                that.getAppListReduced();
                // that.priceTotal(this.curSelectedId);
              }
              //当列表全选时全选按钮高亮
              if(that.curSelectedId.length == that.cartList.length) {
                that.isSelectedAll = true;
              }else {
                that.isSelectedAll = false;
              }
              //购物车ICON微标变化
              that.cartListLen = that.cartList.length;
              that.$store.commit('UPDATE_GoodsCount', data.var.totalCount);
              setStore('_saveCartCount',data.var.totalCount);
            }else{
              if(data.code == "NO_LOGIN" || data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                //没有登录情况下获取本地缓存已经选择的商品详情数组
                that.useCache = true;
                var _cartDetailsData = getStore('_cartDetailsData',true);
                if(typeof(_cartDetailsData) ==='boolean'){
                  that.cartList = [];
                }else{
                  that.cartList = _cartDetailsData;
                }
                if(that.cartList.length == 0 ) {
                  that.show = false;
                  that.tips1 = true;
                }else {
                  that.show = true;
                  that.getAppListReduced();
                  // that.priceTotal(that.curSelectedId);
                }
                //当列表全选时全选按钮高亮
                if(that.curSelectedId.length == that.cartList.length) {
                  that.isSelectedAll = true;
                }else {
                  that.isSelectedAll = false;
                }
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = false;
            that.tips2 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      priceTotal(selectArr) {
        //购物车列表价格计算
        let orgPrice = 0;
        let pointPrice = 0;
        let point = 0;
        let that = this;
        let _reducedTotalPrice = 0;
        for(var i = 0; i < that.cartList.length; i++) {
          // console.log({'index':i});
          //调整代码
            selectArr.forEach(function(item) {
              if(item == that.cartList[i]["id"]) {
                orgPrice += that.cartList[i].orgPrice * that.cartList[i].prodCount;
                // pointPrice += that.cartList[i].pointPrice * that.cartList[i].prodCount;
                pointPrice += that.cartList[i].memberPrice * that.cartList[i].prodCount;
                point += that.cartList[i].memberPoint * that.cartList[i].prodCount;
              }
            });  
            _reducedTotalPrice += that.cartList[i].memberPrice * that.cartList[i].prodCount;
        }
        that.orgPriceCount = parseInt(orgPrice.toFixed(2).split('.')[1])>0?orgPrice.toFixed(2):orgPrice;
        that.orgPointPriceCount = parseInt(pointPrice.toFixed(2).split('.')[1])>0?pointPrice.toFixed(2):pointPrice;
        that.orgPointCount = point;
        that.reducedTotalPrice = that.orgPointPriceCount==0 ? _reducedTotalPrice : that.orgPointPriceCount;
        that.setReduced();
      },
      //设置满减送等级
      setReduced(){
        for(var i=0;i<this.appListReduced.length;i++){
          var item = this.appListReduced[i];
          var _threshold = item.threshold;
          if(i==0 && i==(this.appListReduced.length-1)){//第一级满减送等级且只有一级
            if(this.reducedTotalPrice < _threshold){//总价小于第一个满减送等级
              this.reducedMess = '还差'+(_threshold-this.reducedTotalPrice)+'元就可以参加满'+_threshold+'元减'+item.reducedMoney+'元活动啦~';
              this.isReducedFull = true;
            }else if(this.reducedTotalPrice==_threshold){//总价等于当前满减送等级
                this.reducedMess = '已享受满'+_threshold+'元减'+item.reducedMoney+'元活动';
                this.isReducedFull = false; 
            }
          }else if(i == (this.appListReduced.length-1)){//最后一等级
            if(this.reducedTotalPrice > _threshold || this.reducedTotalPrice == _threshold){
              this.reducedMess = '已享受满'+_threshold+'元减'+item.reducedMoney+'元活动';
              this.isReducedFull = false;
            }
          }else if(i==0){//第一级满减送等级
            if(this.reducedTotalPrice < _threshold){//总价小于第一个满减送等级
              this.reducedMess = '还差'+(_threshold-this.reducedTotalPrice)+'元就可以参加满'+_threshold+'元减'+item.reducedMoney+'元活动啦~';
              this.isReducedFull = true;
              break;
            }else if(_threshold< this.reducedTotalPrice && this.reducedTotalPrice< this.appListReduced[i+1].threshold){
              //总价大于、等于当前满减送等级且小与下一等级
              this.reducedMess = '已享受"满'+_threshold+'元减'+item.reducedMoney+'元"'+'，还差'+(this.appListReduced[i+1].threshold-this.reducedTotalPrice)+'元就可减'+this.appListReduced[i+1].reducedMoney+'元啦';
              this.isReducedFull = true;
              break;
               
            }
          }else{//中间等级
            //总价等于当前满减送等级并且小于下一等级
            if(_threshold <= this.reducedTotalPrice && this.reducedTotalPrice< this.appListReduced[i+1].threshold){
              //当前满减送等级不是最高等级
              this.reducedMess = '已享受"满'+_threshold+'元减'+item.reducedMoney+'元"'+'，还差'+(this.appListReduced[i+1].threshold-this.reducedTotalPrice)+'元就可减'+this.appListReduced[i+1].reducedMoney+'元啦';
              this.isReducedFull = true;
              break;
            }
          }
        }
      },
      newCartArr() {
        //获取新的购物车列表id数组
        let _Arr = [];
        for(let i = 0; i < this.cartList.length; i++) {
          _Arr.push({'prodId': this.cartList[i]['id'], 'prodCount': this.cartList[i]['prodCount']});
        }
        return _Arr;
      },
      saveCartList() {
        //异步提交保存购物车当前数据，主要用于保存购物车已修改数据时使用
        let that = this;
        that.optsAddToBasket.prodIds = that.newCartArr();
        //缓存购物车数据
        setStore("_saveCart", that.optsAddToBasket.prodIds);
        that.$http.post(that.apiAddToBasketUrl+'&shopsid='+window.shopsid,that.optsAddToBasket)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK" || data.code == "NO_DATA") {
              /*//缓存购物车数据
              let _saveCart = JSON.stringify(that.optsAddToBasket.prodIds);
              localStorage.setItem("_saveCart", _saveCart); //购物车添加缓存*/
            }else if(data.code == "NO_LOGIN" || data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
              //缓存商品详情列表数据
              setStore("_cartDetailsData", that.cartList);
              //缓存购物车数据
              setStore("_saveCart", that.optsAddToBasket.prodIds);//购物车添加缓存
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          });
      },
      //获取满减送列表
      getAppListReduced() {
         let that = this;
          that.$http.post(that.apiAppListReduced)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.appListReduced = data['var'].preferentialSets;
              //计算总价
              that.priceTotal(that.curSelectedId);
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                // that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                //   that.$router.push('/login');
                // }});
              }else if(data.code == "NO_LOGIN"){
                // that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                //   that.$router.push('/login');
                // }});
              }else if(data.code != "REDUCED_Q_RECORD_NULL"){
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
      newSelectCartArr() {
        //获取新选择的购物车列表id数组
        let _Arr = [];
        for(let i = 0; i < this.curSelectedId.length; i++) {
          for(let j =0; j < this.cartList.length; j++) {
            if(this.curSelectedId[i] == this.cartList[j]['id']) {
              _Arr.push({'prodId': this.cartList[j]['id'], 'prodCount': this.cartList[j]['prodCount']});
            }
          }
        }
        return _Arr;
      },
      buy(isContinue) {

        this.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_CART_BUY});

        //点击购买按钮触发相关数据请求逻辑
        let that = this;
        that.optsAddToOrder.prodIds = that.newSelectCartArr();
        if(that.optsAddToOrder.prodIds.length == 0) {
          that.$dialog.notify({mes: '您还未添加购买选项', icon: 'error', timeout: 1000});
          return;
        }
        
        that.$dialog.loading.open('获取数据中...');
        that.$http.post(that.apiAddToOrderUrl+'&shopsid='+window.shopsid,that.optsAddToOrder)
          .then((response) => {
            const data = response.data;
             
            if(data.code == "S_OK") {
              removeSessionStore("_lastProdId");
              that.cartSave = false;
              //跳去订单详情
              that.$router.push({
                  path: '/orderdetails',
                  query: {
                    source: 1,
                    oid: data.var
                  }
              });
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "USER_NOT_MEMBER"){
                that.toast3 = true;
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },100);
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },100);
          });
      }
    },
    beforeDestroy() {
      //在页面销毁时（退出时）保存购物车数据
      if(this.cartSave) {
        this.saveCartList();
        this.run(2);
      }
    }
}
</script>
