<template>
  <div>
    <section class="ck-doc skillProduct newProduct">
      <div v-if="!tips1" class="cl">

        <div class="ck-search home-search-bar">
          <router-link to="/search">
            <div class="inner">
              <i class="iconfont">&#xe602;</i>
              <span>请输入搜索的商品名</span>
            </div>
          </router-link>
        </div>
        <article class="content">
         <div class="goods">

    <!-- 商品类型选项 -->
    <div style="overflow-y: hidden;">
      <div class="goods_type" :style="{width: width + 'rem'}">
        <input type="button" class="type-button" :class='{select: select=="-1"}' value="全部" v-on:click='sortBtn("-1")'>
        <input type="button" class="type-button" v-for='(item,index) in sortList' :class='{select: select==item.id}' :value="item.name" v-on:click='sortBtn(item)'>
      </div>
    </div>
    
      <div class="goods_list" v-for='(product,index) in skillProduct'>
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
              <!-- <span class="list-del-price">￥{{product.orgPrice}}</span> -->
             <!-- <span style="color: red;font-size: 0.48rem;">{{val.salePrice}}</span>&nbsp; -->
             <!-- 商品原价 -->
             <span class="goods_price">原价:&yen;{{product.orgPrice}}</span>
             </p>
          </div>

          <!-- 商品活动/销量 -->
          <div style="margin-top: 0.30rem;padding-left: 0.15rem;display: flex;">
            <!-- 活动 -->
            <div class="goods-activity">
              <span v-if="product.memberPrice-product.pointPrice>0" style="color: #ff3d3d;border-color: #ff3d3d;">积分兑现金</span>
              <span v-if="couponList.length>0" style="color: #628bf5;border-color: #628bf5;">优惠卷</span>
              <span  style="color: #ff9a00;border-color: #ff9a00;">会员活动</span>
            </div>

            <div class='goods-sell'>月售999</div>
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

    </section>
  </div>
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
          imgBaseUrl: window.imgBaseUrl,
          couponList:[],
          sortList:[],
          source:1,
          opt:{
            curPage:1,
            pageSize:6,
            sortId:'',
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
      if(sortId){
        this.opt.sortId = sortId;
        this.select = sortId;
      }
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
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_GOOD});
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
      fectchQueryFun(result){
        this.skillProduct = result;
      }
    },
    beforeDestroy() {

    }
}
</script>
