<template>
    <section class="ck-doc skillProduct newProduct">
      <div v-if="!tips1" class="cl">
        <div class="f_top">
          <div class="f_content">
            <div class="ck-search home-search-bar">
              <router-link to="/search">
                <div class="inner">
                  <i class="iconfont">&#xe602;</i>
                  <span>请输入搜索的商品名</span>
                </div>
              </router-link>
            </div>
            <div class="sort_li">
              <div style="overflow-y: hidden;">
                <div class="goods_type" :style="{width: width + 'rem'}">
                  <input type="button" class="type-button" :class='{select: select=="-1"}' value="全部商品" v-on:click='sortBtn("-1")'>
                  <input type="button" class="type-button" v-for='(item,index) in sortList' :class='{select: select==item.id}' :value="item.name" v-on:click='sortBtn(item)'>
                </div>
              </div>
            </div>
          </div>
        </div>
        <article class="content">
          <div class="goods">
            <div class="goods_list" v-for='(product,index) in skillProduct'  @click="detailsGo(product.id,product.activityId,product.serveCode,product.prodSource)">
              <div>
                <img class="goods-img" :src="imgBaseUrl+product.prodPic">
              </div>
              <div>
                <div>
                  <p class="goods-title">{{product.prodTitle}}</p>
                  <!-- 价格 -->
                  <p class="goods-salePrice">
                    <!-- 单独设置￥颜色和大小 -->
                   <span style="color: red;font-size: 0.32rem;">&yen;</span>
                   <!-- 设置出售价格字体大小 -->
                   <span class="s_price" v-if="product.skipe">{{product.skipePrice}}</span>
                    <span class="s_price" v-else-if="product.restrict">{{product.discountPrice}}</span>
                    <span class="s_price" v-else>{{product.memberPrice}}</span>
                   <!-- 商品原价 -->
                   <span class="goods_price">原价:&yen;{{product.orgPrice}}</span>
                   </p>
                </div>

                <!-- 商品活动/销量 -->
                <div style="margin-top: .25rem;padding-left: 0.15rem;display: flex;">
                  <!-- 活动 -->
                  <div class="goods-activity">
                    <span style="color: #ff3d3d;border-color: #ff3d3d;">积分兑现金</span>
                    <span v-if="product.couponId" style="color: #628bf5;border-color: #628bf5;">优惠卷</span>
                    <span  style="color: #ff9a00;border-color: #ff9a00;">会员活动</span>
                  </div>
                  <div class='goods-sell' v-if="product.saleNumber || product.saleTotal">月售{{product.saleNumber+product.saleTotal}}</div>
                </div>
              </div>
            </div>
          </div>
        </article>
      </div>

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
import {setShopsId,getSessionStore,removeSessionStore,setSessionStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
import ajaxModel from '../utils/ajaxModel';
export default {
    data() {
      return {
        tips1: false, 
        title: '商品',
        disabled: true,
        imgBaseUrl: window.imgBaseUrl,
        couponList:[],
        sortList:[],
        source:1,
        orgCode: window.orgCode,
        opt:{
          curPage:1,
          pageSize:6,
          sortId:'',
          identifierCode:'',
          isDesc:'',
          name:'',
          searchKey:''
        },
        width:null,
        select:'-1',
        skillStatu: {},//秒杀的状态
        skillProduct:[]//秒杀商品
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    beforeMount() {//fetchList('m3',menu.code,menu.name)
      //挂载完毕初始化页面数据
      //获取路由中的搜索关键字
      let _searchKey = this.$route.query.searchKey;
      if(_searchKey != null || _searchKey != 'undefined') {
        this.opt.searchKey = _searchKey;
      }
      let sortId = this.$route.query.pId;
      this.opt.sortId = sortId == '-1'?'':sortId;
      this.select = sortId;
      ajaxModel.queryProductList(this,this.fectchQueryFun,this.opt,1);
      this.markInit();
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
        ajaxModel.findMenu(this,this.fecthSortFun);
        window.addEventListener('scroll', this.menu);
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_GOOD});
      },
      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
            this.loadList();
        }
      },
      detailsGo(id,activityId,serveCode,prodSource) {
          //跳去详情页
          // if(serveCode == '' || prodSource !='4'){
          // if(serveCode == ''){
          //   this.$router.push({
          //     path: '/details',
          //     query: {
          //       pid: id,
          //       psource:this.source,
          //       activityId:activityId
          //     }
          //   });
          //   setSessionStore("goods_nav_menu",{'id':this.m3curId,'navTitle':this.navTitle,'source':this.source});
          // }else{
          //   var str = serveCode.split('_')[0];
          //   this.$router.push({
          //     path: '/serviceDetail',
          //     query: {
          //       'pid': id,
          //       'serveCode':str,
          //       activityId:activityId,
          //       'psource':this.source
          //     }
          //   });
          // }

          if(serveCode && serveCode.split('_')[0] =='GLASSSUPPORT'){
            this.$router.push({path: '/serviceDetail'});
          }else{
            setSessionStore("goods_nav_menu",{'id':this.m3curId,'navTitle':this.navTitle,'source':this.source});
            this.$router.push({
              path: '/details',
              query: { 
                pid: id,
                psource:this.source,
                activityId:activityId
              }
            });
          }
      },
      //菜单点击
      sortBtn(pOpt){
        if(pOpt == '-1'){
          this.select = '-1';
          this.opt.sortId = '';
        }else{
          this.select = pOpt.id;
          this.opt.sortId = pOpt.id;
        }
        this.opt.searchKey = '';
        this.opt.curPage = 1;
        ajaxModel.queryProductList(this,this.fectchQueryFun,this.opt,1);
      },
      fecthSortFun(result){
        //获取首页数据
        this.sortList = result.sort;
      },
      //秒杀商品跳去详情页
      goTo(id,activityId,serveCode,prodSource) {
        //跳去详情页
        if(serveCode == ''){
          this.$router.push({
            path: '/details',
            query: {
              pid: id,
              psource:prodSource,
              activityId:activityId
            }
          });
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
      loadList() {
        //分页加载页面数据
        let that = this;
        that.opt.curPage++;
        ajaxModel.queryProductList(this,(result)=>{
          for(let plist of result) {
            that.skillProduct.push(plist);
          }
        },this.opt,1);
      },
      fectchQueryFun(result){
        this.skillProduct = result;
      }
    },
    beforeDestroy() {

    }
}
</script>
