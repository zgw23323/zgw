<template>
  <section class="c-doc evaluate">
    <article class="e_content">
    	<div class="e_store">
	        <div class="store_logo">&nbsp;</div>
	        <p>深圳福田MG</p>
	        <div class="xin i_flex_center">
	          <span :class="{'x_cur':item.isBtn,'no':!item.isBtn}" v-if="applyOpt.starCount!= 0" v-for="item,index in xinList">&nbsp;</span>
	          <span :class="{'x_cur':item.isBtn,'no':!item.isBtn}" v-if="applyOpt.starCount == 0" v-for="item,index in xinList" @click="xinBtn(index)">&nbsp;</span>
	        </div>
        </div>
        <div class="e_label i_flex">
        	<span class="l_cur">很专业</span>
        	<span>速度很快</span>
        	<span>服务很棒</span>
        	<span>很专业</span>
        	<span>速度很快</span>
        	<span>服务很棒</span>
        	<span>很专业</span>
         </div>
        <div class="s_text">
          <textarea type='text' v-model="commentDesc" placeholder="您还有什么肺腑之言呢？"></textarea>
        </div>
        <div class="b_btn">提交</div>
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
        title: '抢购成功',
        isShow:false,
        imgBaseUrl: window.imgBaseUrl,
        xinList:[
          {id:0,isBtn:false},{id:0,isBtn:false},{id:0,isBtn:false},{id:0,isBtn:false},{id:0,isBtn:false}
        ],
        type:'',
        applyOpt:{
          detailId:'1',
          starCount:0,
          commentDesc:''
        },
        commentDesc:'',
        orderData:{}
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_ACTIONORDERINFO});
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
       
      run() {
        this.timer = setInterval(() => {
          this.getCodeStatu()
        }, 2000);
      },
      //星星点击
      xinBtn(n){
        this.xinList[n].isBtn = !this.xinList[n].isBtn;
        if(this.xinList[n].isBtn){
          for(var i=0;i<n;i++){
            this.xinList[i].isBtn = true;
          }
        }else{
          for(var i=n;i<this.xinList.length;i++){
            this.xinList[i].isBtn = false;
          }
        }
      }
    },
    destroyed() {
      clearInterval(this.timer);
    }
  }
                
</script>
<style lang="less">
  @import "../../styles/common/action.less";
</style>