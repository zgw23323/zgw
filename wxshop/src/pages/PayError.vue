<template>

<!--	<transition name="page">-->
		<section class="ck-doc mybill">
			<div class="tipsPage">
		      <div class="tipsBox">
		        <div class="tipStatus" data-type="ordererror"></div>
		        <p style="color:#db2c1d">很抱歉，支付失败！{{errCode}}</p>
		      </div>
		    </div>

      <div class="mybill-ft fixed">
        <div class="inner">
          <p class="p1">{{shop.name}}</p>
          <p class="p2">{{shop.telphone}} {{shop.address}}</p>
        </div>
      </div>
		</section>
<!--	</transition>-->

</template>

<script type="text/babel">
import {LOGCODE} from '../utils/logCode';
export default {
	data() {
		return {
			show: false,
			title: '支付失败',
      shop: {
        name: '',
        telphone: '',
        address: ''
      },
      errCode:''
		}
	},
	watch: {
      '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
    	this.markInit();
    	setTimeout(()=>{
      		this.show = true;
      	},300);
    },
	methods: {
	  markInit() {
      //初始化页面标题
	  	document.title = this.title;
      //获取门店信息
      this.shop = window.shop;

      this.errCode = this.$route.query.err || '';

      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_PAYERR});
	  }
	}

}
</script>
