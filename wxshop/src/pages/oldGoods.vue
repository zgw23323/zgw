<template>
  <div>
    <section class="ck-doc goods">
      <div v-if="!tips1" class="cl">
        <div class="filterbar-wrap" v-if="!tips1">
          <div class="filter-bar">
            <div class="ck-search">
              <router-link to="/search">
                <i class="iconfont">&#xe602;</i>
              </router-link>
            </div>
            <ul class="sort-tab">
              <li class="sort" @click="dropList(1)" :class="{'current': dropList1, 'selected': 1 == m1navcurId ? true : false}">{{navTitle1}}<i class="arrow"></i></li>
              <li class="sort" @click="dropList(2)" :class="{'current': dropList2, 'selected': 2 == m1navcurId ? true : false}">综合排序<i class="arrow"></i></li>
              <li class="sort common-sort" @click="dropList(3)" :class="{'current': dropList3, 'selected': 3 == m1navcurId ? true : false}"><span>{{navTitle}}</span><i class="arrow"></i></li>
            </ul>

            <div id="drop-list1" class="drop-list" v-if="dropList1">
               <ul class="drop-list-first">
                <li :class="'all' == m1curId ? 'current' : ''" @click="fetchSubMenu('-1','all')"><p>全部<i class="iconfont icon_left_only" v-if="m1SelectCurId=='all'">&#xe72e;</i></p></li>
                <li :class="'prodSource' == m1curId ? 'current' : ''" @click="fetchSubMenu('4','prodSource')"><p>权益服务<i class="iconfont icon_left_only" v-if="m1SelectCurId=='prodSource'">&#xe72e;</i></p></li>
                <li :class="menu.id == m1curId ? 'current' : ''" v-for="(menu,index) in menuData.sort" @click="fetchSubMenu(index,menu.id)">
                  <p>{{menu.name}}<i class="iconfont icon_left_only" v-if="m1SelectCurId==menu.id">&#xe72e;</i></p>
                  <ul class="drop-list-chil" v-if="menu.id == m1subcurId&&menu.children.length>0">
                    <li v-for="sub in menu.children" @click="fetchList('m1',sub.id,'msub',$event)"><div>{{sub.name}}</div><div><i class="iconfont" v-if="sub.id==m1SelectCurId">&#xe72e;</i></div></li>
                  </ul>
                </li>
              </ul>
            </div>

            <div class="drop-list" v-if="dropList2">
              <ul class="drop-list-middle">
                <li :class="{'reverse': m2reverse1 ? true : false, 'selected': 4 == m2curId ? true : false}" @click="fetchList('m2','4')">综合排序<span class="arrow"></span><i class="iconfont">&#xe72e;</i></li>
                <li :class="{'reverse': m2reverse1 ? true : false, 'selected': 1 == m2curId ? true : false}" @click="fetchList('m2','1')">按时间<span class="arrow"></span><i class="iconfont">&#xe72e;</i></li>
                <li :class="{'reverse': m2reverse2 ? true : false, 'selected': 2 == m2curId ? true : false}" @click="fetchList('m2','2')">按销量<span class="arrow"></span><i class="iconfont">&#xe72e;</i></li>
                <li :class="{'reverse': m2reverse3 ? true : false, 'selected': 3 == m2curId ? true : false}" @click="fetchList('m2','3')">按金额<span class="arrow"></span><i class="iconfont">&#xe72e;</i></li>
              </ul>
            </div>

            <div class="drop-list" v-if="dropList3">
             <ul class="drop-list-middle" v-if="source=='2' || source == '3'">
                <li :class="2 == m3curId ? 'selected' : ''" @click="fetchList('m3',2,'秒杀')">秒杀<i class="iconfont">&#xe72e;</i></li>
                <li :class="3 == m3curId ? 'selected' : ''" @click="fetchList('m3',3,'限时折扣')">限时折扣<i class="iconfont">&#xe72e;</i></li>
              </ul>
              <ul class="drop-list-middle" v-else>
                <li :class="m3curId == '' ? 'selected' : ''" @click="fetchList('m3','','全部')">全部<i class="iconfont">&#xe72e;</i></li>
                <li :class="menu.code == m3curId ? 'selected' : ''" v-for="menu in menuData.mark" @click="fetchList('m3',menu.code,menu.name)">{{menu.name}}<i class="iconfont">&#xe72e;</i></li>
              </ul>
             
            </div>

          </div>
        </div>

        <div class="filterbar-mark" v-if="mark" @click="closeDropList"></div>

        <div class="goods-list goods-top" style="padding-bottom: .2rem;">

          <div v-infinite-scroll="loadList" infinite-scroll-disabled="disabled" infinite-scroll-distance="10">
            <yd-list theme="3" slot="list">
              <div v-for="product in productList" class='goods-par-dom' @click="detailsGo(product.id,product.activityId,product.serveCode,product.prodSource)">
                 <!--其它商品-->
                <yd-list-item v-if="(product.serveType==0&&product.serveCount==0) || product.joinActivity">        
                  <div class="goodsimg imgLazy" slot="img" v-lazy:background-image="imgBaseUrl+product.prodPic"></div>
                  <span slot="title">{{product.prodTitle}}</span>
                  <yd-list-other slot="other">
                    <!-- <p class="goods-price">￥{{product.memberPrice}}<span class="list-del-price">￥{{product.orgPrice}}</span></p> -->
                    <p class="goods-price">
                      <span v-if="product.skipe && source != 4 && source != 1">￥{{product.skipePrice}}</span>
                      <span v-else-if="product.restrict && source != 4 && source != 1">￥{{product.discountPrice}}</span>
                      <span v-else>￥{{product.memberPrice}}</span>
                      <span class="list-del-price">￥{{product.orgPrice}}</span>
                    </p>
                    <span class="goods-memberPoint" v-if="product.springProdId">一元领取代金券</span>
                    <!--参与活动的商品没有积分抵扣金额-->
                    <span class="goods-memberPoint" v-else-if="!product.restrict && !product.skipe && product.memberPrice-product.pointPrice>0">{{product.memberPoint}}积分可兑换{{product.memberPrice-product.pointPrice}}元现金</span>
                    <!--活动商品状态-->
                    <!--限时折扣已开始-->
                    <div class='action-status action-ing' v-if="product.restrict && product.status==3">抢购中</div>
                    <!--秒杀已开始-->
                    <div class='action-status action-ing' v-if="product.skipe && product.status==3">秒杀中</div>
                    <div class='action-status action-end' v-if="(product.skipe || product.restrict)&& product.status==4">已结束</div>
                    <div class='action-status action-no' v-if="product.skipe && product.status==1">预约中</div>
                    <div class='action-status action-no' v-if="product.restrict && product.status==1">未开始</div>
                    <!--限时购logo-->
                    <!-- <div class="action_logo_b action_logo_common" v-if="product.restrict==true"></div> -->
                    <!--限时折扣logo-->
                    <div class="action_logo_z action_logo_common" v-if="product.restrict==true && source != 4 && source != 1"></div>
                     <!--秒杀logo-->
                    <div class="action_logo_m action_logo_common" v-if="product.skipe==true && source != 4 && source != 1"></div>
                  </yd-list-other>
                </yd-list-item>
                 <!--权益服务-->
                <yd-list-item v-else>        
                  <div class="goodsimg imgLazy" slot="img" v-lazy:background-image="imgBaseUrl+product.prodPic"></div>
                  <span slot="title" v-if="product.serveType==1">{{product.prodName}} (无限次)</span>
                  <span slot="title" v-else>{{product.prodName}} ({{product.serveCount+'次'}})</span>
                  <!-- <span slot="title">{{product.prodName}}</span> -->
                  <yd-list-other slot="other">
                    <!-- <p class="goods-price">￥{{product.memberPrice}}<span class="list-del-price">￥{{product.orgPrice}}</span></p> -->
                    <p class="goods-price">
                      <span v-if="product.skipe && source != 4 && source != 1">￥{{product.skipePrice}}</span>
                      <span v-else-if="product.restrict && source != 4 && source != 1">￥{{product.discountPrice}}</span>
                      <span class="service_price_mess" v-else>￥{{product.memberPrice}}<span class='s_memberMess'>活动价</span></span>
                      <span class="list-del-price">市场价￥{{product.orgPrice}}</span>
                    </p>
                    <!--活动商品状态-->
                    <!--限时折扣已开始-->
                    <div class='action-status action-ing' v-if="product.restrict && product.status==3">抢购中</div>
                    <!--秒杀已开始-->
                    <div class='action-status action-ing' v-if="product.skipe && product.status==3">秒杀中</div>
                    <div class='action-status action-end' v-if="(product.skipe || product.restrict)&& product.status==4">已结束</div>
                    <div class='action-status action-no' v-if="product.skipe && product.status==1">预约中</div>
                    <div class='action-status action-no' v-if="product.restrict && product.status==1">未开始</div>
                    <!--限时购logo-->
                    <!-- <div class="action_logo_b action_logo_common" v-if="product.restrict==true"></div> -->
                    <!--限时折扣logo-->
                    <div class="action_logo_z action_logo_common" v-if="product.restrict==true && source != 4 && source != 1"></div>
                     <!--秒杀logo-->
                    <div class="action_logo_m action_logo_common" v-if="product.skipe==true && source != 4 && source != 1"></div>
                  </yd-list-other>
                </yd-list-item>
              </div>
            </yd-list>
          </div>

        </div>
      </div>

      <div class="tipsPage" v-if="tips1">
        <div class="tipsBox">
          <div class="tipStatus" data-type="busy"></div>
          <p>亲，网络出问题啦~~</p>
          <div class="btns" @click="reLoad">重新加载</div>
        </div>
      </div>

    </section>

    <ck-nav></ck-nav>
  </div>
</template>

<script type="text/babel">
import {setShopsId,getSessionStore,removeSessionStore,setSessionStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
export default {
    data() {
        return {
          disabled: true,
          mark: false,
          tips1: false,
          doneTipShow: true,
          dropList1: false,
          dropList2: false,
          dropList3: false,
          m2reverse1: false,
          m2reverse2: false,
          m2reverse3: false,
          m1curId: 'all',
          navTitle1:'全部商品',
          navTitle:'通用',//设置通用筛选标题，下拉选择实时更新
          m1SelectCurId:'all',
          m1navcurId: '',
          m1subcurId: '',
          m2curId: '',
          m3curId: '',
          title: '商品',
          serveCode:'',
          source:'',//查询商品来源：1=限时折扣，2=秒杀；3=优惠券
          imgBaseUrl: window.imgBaseUrl,
          apiUrl: window.baseUrl+window.appName+'/shopapi/func/product/queryProductList?orgCode='+window.orgCode,
          menuUrl: window.baseUrl+window.appName+'/shopapi/func/sort/findMenu?orgCode='+window.orgCode,
          apiCouponUrl: window.baseUrl+window.appName+'/shopapi/func/coupon/couponProudctList?orgCode='+window.orgCode+'&shopsid='+ window.shopsid,
          opts: {curPage: 1, pageSize: 6},
          productList: [],
          menuData: {},
          menuSubData: []
        }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    beforeMount() {//fetchList('m3',menu.code,menu.name)
      //挂载完毕初始化页面数据
      let pId = this.$route.query.pId;
      let _navTitle = this.$route.query.navTitle;
      var serviceInit = this.$route.query.serviceInit;
      var seachType = this.$route.query.seachType;
      this.serveCode = this.$route.query.serveCode;
      // let sessionGoodsNav = sessionStorage.getItem('goods_nav_menu');
      var sessionGoodsNav = getSessionStore('goods_nav_menu',true);
      if(pId){
        if(pId =='secKillCount'){//秒杀
          this.source = '2';
          this.title = _navTitle;
          this.navTitle = '秒杀';
          // this.fetchList();
          this.fetchList("m3",2,'秒杀');
        }else if(pId =='restrictCount'){//限时折扣
          this.source = '3';
          this.title = _navTitle;
          this.navTitle = '限时折扣';
          this.fetchList("m3",3,'限时折扣');
        }else if(pId == 'coupon'){//优惠券
          this.source = '4';
          this.fetchList();
        }else if(seachType == 'sort'){//首页分类点进来的
          this.source = '1';
          this.navTitle = _navTitle ? _navTitle :this.navTitle;
          this.m1curId = pId;
          this.m1SelectCurId = pId;
          this.fetchList('m1',pId,'mparent');
        }else{//普通商品
          this.source = '1';
          this.navTitle = _navTitle ? _navTitle :this.navTitle;
          this.fetchList('m3',pId,this.navTitle);
        }
      //判断缓存是否有数据,如果有查询缓存中的数据      
      }else if(typeof('sessionGoodsNav')==='boolean'){
        // var  goodsNavInfo = JSON.parse(sessionGoodsNav);
        if(goodsNavInfo.source != '1'){
          this.source = goodsNavInfo.source;
          this.fetchList();
        }else{
          this.source = '1';
          this.fetchList('m3',goodsNavInfo.id,goodsNavInfo.navTitle);
        }
      }else{//判断是否是权益服务过来的列表
        this.source = '1';
        if(serviceInit){
          this.navTitle1 = '权益服务';
          this.fetchList('m1','prodSource','msub');
        }else{
          this.fetchList();
        }
      }
      //清除菜单缓存
       // sessionStorage.removeItem('goods_nav_menu');
       removeSessionStore('goods_nav_menu');
       this.markInit();
    },
    methods: {
        reLoad() {
          //重载页面
          this.$router.go(0);
          this.markInit();
        },
        detailsGo(id,activityId,serveCode,prodSource) {
          //跳去详情页
          // if(serveCode == '' || prodSource !='4'){
          if(serveCode == ''){
            this.$router.push({
              path: '/details',
              query: {
                pid: id,
                psource:this.source,
                activityId:activityId
              }
            });
            setSessionStore("goods_nav_menu",{'id':this.m3curId,'navTitle':this.navTitle,'source':this.source});
          }else{
            var str = serveCode.split('_')[0];
            this.$router.push({
              path: '/serviceDetail',
              query: {
                'pid': id,
                'serveCode':str,
                activityId:activityId,
                'psource':this.source
              }
            });
          }
          
        },
        markInit() {
          //初始化页面标题
          document.title = this.title;
          //获取shopsid缓存数据
          setShopsId();
          this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_GOOD});
        },
        closeDropList() {
          //关闭下拉筛选列表
          setTimeout(()=>{
            this.dropList2 = false;
            this.dropList3 = false;
            this.dropList1 = false;
            this.mark = false;
          },200);
        },
        fetchList(c0,c1,c2,c3) {
          //商品列表数据请求方法
          let that = this;          
          that.disabled = true;
          that.productList = [];
          that.opts.sortId = "";
          that.opts.identifierCode = "";
          that.opts.name = "";
          that.opts.searchType = "";
          that.opts.searchKey = "";
          that.opts.isDesc = 1;
          that.m1navcurId = 1;
          if(c0 == "m1") {  //商品分类
            that.m1navcurId = 1;
            that.m2curId= '';
            that.m3curId= '';
            that.m1subcurId = c1;
            that.opts.sortId = c1;
            that.m2reverse1 = false;
            that.m2reverse2 = false;
            that.m2reverse3 = false;
            // that.m1SelectCurId = 'all';
            that.navTitle = '通用';//点别的标签重置通用筛选头部
            if(c2 == 'msub') {
              that.closeDropList();
              that.m1subcurId = that.m1curId;
              that.m1SelectCurId = c1;//修改UI后，存放当前选中项
              that.opts.prodSource = "";
              if(c1 != 'prodSource'){
                c3.cancelBubble = true;
              }
              // console.log({'text2':that.m1SelectCurId,'c22':c2});
              // console.log('event=>', c3);
            }
            if(c1 == 'prodSource'){
              that.opts.serveCode = this.serveCode;
              that.navTitle1 = "权益服务";
              that.opts.sortId = "";
              that.opts.prodSource = "4";
            }else{
              that.opts.serveCode = '';
              that.navTitle1 = "全部商品";
              that.opts.prodSource = "";
            }
          }else if(c0 == "m2") { //排序分类
            that.m1navcurId = 2;
            that.m1curId= 'all';
            that.m1subcurId= '';
            that.m3curId= '';
            that.m2curId = c1;
            that.opts.prodSource = "";
            that.m1SelectCurId ='all';
            that.navTitle1 = "全部商品";
            that.navTitle = '通用';//点别的标签重置通用筛选头部
            that.opts.searchType = c1;
            if(c1 == '1') { //按时间分类
              that.m2reverse2 = false;
              that.m2reverse3 = false;
              that.m2reverse1 = !that.m2reverse1;
              if(that.m2reverse1) {
                that.opts.isDesc = 2;
              }
            }else if(c1 == '2') { //按销量分类
              that.m2reverse1 = false;
              that.m2reverse3 = false;
              that.m2reverse2 = !that.m2reverse2;
              if(that.m2reverse2) {
                that.opts.isDesc = 2;
              }
            }else if(c1 == '3') { //按金额分类
              that.m2reverse1 = false;
              that.m2reverse2 = false;
              that.m2reverse3 = !that.m2reverse3;
              if(that.m2reverse3) {
                that.opts.isDesc = 2;
              }
            }
            if(c2 != 'first') {
              that.closeDropList();
            }else {
              that.m2reverse1 = !that.m2reverse1;
            }
          }else if(c0 == "m3") { //筛选分类
            that.m1navcurId = 3;
            that.m1curId= 'all';
            that.m1subcurId= '';
            that.m2curId = '';
            that.m3curId = c1;
            that.opts.prodSource = "";
            that.m1SelectCurId ='all';
            that.navTitle1 = "全部商品";
            that.opts.identifierCode = c1;
            that.opts.name = c2;
            that.m2reverse1 = false;
            that.m2reverse2 = false;
            that.m2reverse3 = false;
            that.navTitle = c2;
            if(c3 != 'first') {
              that.closeDropList();
            }
            if(c2 == '秒杀'){
              that.source = '2';
            }
            if(c2 == '限时折扣'){
              that.source = '3';
            }
          }
          //获取路由中的搜索关键字
          let _searchKey = this.$route.query.searchKey;
          if(_searchKey != null || _searchKey != 'undefined') {
            that.opts.searchKey = _searchKey;
          }
          that.$dialog.loading.open('加载数据中...');
          //缓存选择菜单按钮信息
          if(that.source=='4'){
            var _url = that.apiCouponUrl + '&fromSource=' + that.source;
            // that.opts.couponId = sessionStorage.getItem('couponId');
            that.opts.couponId = getSessionStore('couponId');
          }else{
            var _url = that.apiUrl +'&fromSource=' + that.source; 
          }
          that.$http.post(_url,that.opts)
            .then((response) => {
              const data = response.data;
              that.tips1 = false;
              if(data.code == "S_OK") {
                that.disabled = false;
                that.opts.total = data.total;
                if(data.total > 1) {
                  that.doneTipShow = true;
                }else {
                  that.doneTipShow = false;
                }
                let productArr = [];
                for (let plist of data.var) {
                  productArr.push(plist);
                }
                that.productList = productArr;
              }else{
                if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                  that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/login');
                  }});
                }else if(data.code == "NO_LOGIN") {
                  that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/login');
                  }});
                }else{
                  that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
                }
                
              }
              setTimeout(()=> {
                that.$dialog.loading.close();
              },300);
            },(response) => {
              that.tips1 = true;
              that.$dialog.notify({mes: '网络不好，请稍后再试！', icon: 'error', timeout: 1200});
              setTimeout(()=>{
                that.$dialog.loading.close();
              },300);
            });
        },
        fetchMenu(m) {
          //获取下拉筛选的菜单类名
          let that = this;
          if(m == 'm2') {
            if(this.m2curId == '') {
              this.fetchList('m2','4','first');
            }
            return
          }
          that.opts.serveCode = '';
          that.$http.post(that.menuUrl)
            .then((response) => {
              const data = response.data;
              if(data.code == "S_OK") {
                that.menuData = data.var;
                if(m == 'm1') {
                  console.log({'type':'fetchMenu','text3':that.m1SelectCurId,'c223':that.m1curId,'m1subcurId':that.m1subcurId});
                  if(that.m1curId == "" || that.m1curId == "all") {
                    that.m1curId = "all";
                    that.m2curId = '';
                    that.m1SelectCurId = 'all';
                    that.menuSubData = '';
                    that.fetchList();
                  }
                }else if(m == 'm3') {
                  if(that.m3curId == '') {
                    that.m3curId = '';
                    that.fetchList('m3','','全部','first');
                  }
                }
              }else {
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }
            },(response) => {
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            });
        },
        fetchSubMenu(i,id) {
          //商品分类中，获取子菜单的类名
          let menuSubs = '';
          this.serveCode = '';
          this.opts.curPage = 1;
          if(i == '-1') {
            this.m1curId = 'all';
            this.m1SelectCurId = 'all';//修改UI后，存放当前选中项
            id = '';
          }else if(id =='prodSource') {
            this.m1curId = id;
            this.m1SelectCurId = id;
          }else{
            menuSubs = this.menuData.sort[i].children;
            this.m1curId = id;
            this.m1SelectCurId = '';
          }
          this.menuSubData = menuSubs;
          if(menuSubs.length == 0) {
            this.closeDropList();
            if(i !='-1'){
              this.m1SelectCurId = id;
            }
          }
          if(id=="prodSource"){
            this.fetchList('m1',id,'msub');
          }else{
             this.fetchList('m1',id,'mparent');
          }
        },
        loadList() {
          //分页加载页面数据
          let that = this;
          if(!that.disabled) {
            that.opts.curPage++;
          }else {
            return
          }
          if(that.opts.curPage > that.opts.total) {
            /* 所有数据加载完毕 */
            that.disabled = true;
            that.opts.curPage =  that.opts.total;
            //window.$yduiBus.$emit('ydui.infinitescroll.loadedDone');
            return;
          }
           //缓存选择菜单按钮信息
          if(that.source=='4'){
            var _url = that.apiCouponUrl + '&fromSource=' + that.source;
            // that.opts.couponId = sessionStorage.getItem('couponId');
            that.opts.couponId = getSessionStore('couponId');
          }else{
            var _url = that.apiUrl +'&fromSource=' + that.source; 
          }
          that.$http.post(_url,that.opts)
            .then((response) => {
              const data = response.data;
              that.disabled = false;

              if(data.code == "S_OK") {
                for(let plist of data.var) {
                  that.productList.push(plist);
                }
              }else {
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }

              /* 单次请求数据完毕 */
              //window.$yduiBus.$emit('ydui.infinitescroll.finishLoad');

            },(response) => {
              that.disabled = false;
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            });
        },
        dropList(num) {
          //展示下拉列表
          let _searchKey = this.$route.query.searchKey;
          this.opts.curPage = 1;
          if(_searchKey != null || _searchKey != undefined) {
            this.opts.searchKey = "";
            this.$router.push('/goods');
          }
          //禁止网页滚动
          //document.body.style.overflow = "hidden";
          if(num == 1) {  //商品分类
            this.fetchMenu('m1');
            this.dropList2 = false;
            this.dropList3 = false;
            this.dropList1 = !this.dropList1;
          }else if(num == 2) {  //排序分类
            this.fetchMenu('m2');
            this.dropList1 = false;
            this.dropList3 = false;
            this.dropList2 = !this.dropList2;
            this.serveCode = '';
          }else if(num == 3) { //筛选分类
            this.fetchMenu('m3');
            this.dropList1 = false;
            this.dropList2 = false;
            this.dropList3 = !this.dropList3;
            this.serveCode = '';
          }
          if(this.dropList1 == true || this.dropList2 == true || this.dropList3 == true) {
            this.mark = true;
          }else {
            this.mark = false;
          }
        }
    },
    beforeDestroy() {

    }
}
</script>
