<template>
<footer class="nav-footer">
  <div class="nav-ft-tabbar cl">
      <router-link to="/home">
        <div class="i-icon i-home"></div>
        <div class="i-name">首页</div>
      </router-link>
      <router-link :to="{path: 'goods', query: {pId: '-1'}}">
        <div class="i-icon i-goods"></div>
        <div class="i-name">商品</div>
      </router-link>
      <router-link to="/cart">
        <div class="i-icon i-cart"></div>
        <div class="i-name">购物车</div>
        <yd-badge type="danger" v-text="goodsCount" v-if="badgeShow"></yd-badge>
      </router-link>
      <router-link to="/personal">
        <div class="i-icon i-personal"></div>
        <div class="i-name">个人中心</div>
      </router-link>
  </div>
</footer>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {getStore} from '../../../utils/assist';
  export default {
    name: 'ck-nav',
    data() {
      return {
        badgeShow: false
        // goodsCount: 0
      }
    },
    props: {
      count: ''
    },
    watch: {
      goodsCount() {  //深度监商品数据,用于显示购物车ICON红色微标
        if(this.goodsCount > 0 ) {
          this.badgeShow = true;
        }else {
          this.badgeShow = false;
        }
      },
      count() { //深度监商品传入数据
        // this.goodsCount = this.count;
      },
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
    },
    computed: {
      ...mapState([
        'goodsCount'
      ])
    },
    methods: {
      markInit() {
        //获取购物车缓存数据
        // let _saveCartCount = JSON.parse(localStorage.getItem("_saveCartCount"));
        let _saveCartCount = getStore('_saveCartCount');
        // if(_saveCartCount == null || _saveCartCount == 'undefined' || _saveCartCount == ''|| _saveCartCount == 0) {
        if(typeof(_saveCartCount) ==='boolean'){
          // this.goodsCount = 0;
          // let _saveCart = JSON.parse(localStorage.getItem("_saveCart"));
            let _saveCart = getStore("_saveCart",true);
            // if(_saveCart == null || _saveCart == 'undefined' || _saveCart == ''|| _saveCart == 0) {
            if(typeof(_saveCartCount) ==='boolean'){
              this.$store.commit('UPDATE_GoodsCount', 0);
            }else{
              this.$store.commit('UPDATE_GoodsCount', _saveCart.length); 
            }
        }else {
          // this.goodsCount = _saveCart.length;
          this.$store.commit('UPDATE_GoodsCount', _saveCartCount); 
        }
        // that.$store.commit('UPDATE_GoodsCount', this.goodsCount);
        console.log(this.goodsCount+'&&'+_saveCartCount);
        if(this.goodsCount > 0 ) {
          this.badgeShow = true;
        }else {
          this.badgeShow = false;
        }
      }
    }
  }
</script>
