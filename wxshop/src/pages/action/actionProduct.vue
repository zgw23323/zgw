<template>
	<section class="actionProduct action_bg3">
    <article class="a_content">
      <div class="p_content">
        <ul class="i_flex">
          <li v-for="product in productData.shiwuProds">
            <div :class="{'has_b':productData.shiwuProdId && productData.shiwuProdId!=product.prodId}">
              <div class="p_img i_flex_center"><img :src="imgBaseUrl+product.picFileUrl" /></div>
            </div>
            <h3>{{product.prodTitle}}</h3>
              <router-link class="a_btn" v-if="productData.shiwuProdId && productData.shiwuProdId==product.prodId" :to="{path:'actionOrderInfo',query:{id:productData.orderId,serveCode:'product'}}">&nbsp;</router-link>
              <router-link class="a_btn" v-else-if="!productData.shiwuProdId" :to="{path:'actionProductDetail',query:{pid:product.prodId,serveCode:'product'}}">&nbsp;</router-link>
            <div class="a_btn no" v-else>&nbsp;</div>
          </li>
        </ul>
      </div>
      <div class="s_content">
        <div class="s_head"><span>代金券使用有效期至：2018年3月31日</span></div>
        <ul>
          <li class="s_item i_flex" v-for="coupon in productData.couponProds">
            <div class="s_img i_flex_center">
              <div class="r_2"><i :style="{backgroundImage: 'url(' + imgBaseUrl+coupon.picFileUrl + ')'}">&nbsp;</i></div>
              <span class="y_top">&nbsp;</span>
              <span class="y_bottom">&nbsp;</span>
            </div>
            <div class="right">
              <div class="r_1">
                <h2>￥<span>{{coupon.couponPrice}}</span></h2>
                <h3>{{coupon.prodTitle}}</h3>
                <router-link class="s_btn" :to="{path:'actionProductDetail',query:{pid:coupon.prodId,serveCode:'DJSERVICES'}}">&nbsp;</router-link>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </article>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {wxConfig} from '../../utils/wxSDK';
  export default {
    data() {
      return {
        title: '长城车主回家路，有道汽车服务您',
        imgBaseUrl: window.imgBaseUrl,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/springwarm/wxYiyuanProducts?sid='+window.shopsid+'&orgCode='+window.orgCode,
        productData:{}
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchProductData();
      this.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_ACTIONPRODUCT});
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
      },
      fetchProductData() {
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl)
          .then((response) => {
            const data = response.data;
            if (data.code === 'S_OK') {
              this.productData = data.var;
            }
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);

          },(response) => {
            setTimeout(()=>{
                that.$dialog.loading.close();
                that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
              },300);
          });
      }
    }
  }
</script>
<style lang="less">
  @import "../../styles/common/action.less";
</style>