 <template>
  <section class="service drivingServiceDetail serviceDetail details">
    <article class="header">
       <div class="details-hd">
          <div class="glass_GLASSSUPPORT">&nbsp;</div>
        </div>
    </article>
    <article class="content">
      <div class="con_info glassSupport" style="padding: .59rem 0;">
        <div><img src="../../img/serviceImg/detail_26.png" style="width:7.13rem;margin-bottom: .67rem;" /></div>
        <div><img src="../../img/serviceImg/detail_27.png" style="margin-bottom:.57rem;" /></div>
        <div><img src="../../img/serviceImg/detail_28.png" style="width:6.9rem;margin-bottom: .76rem;" /></div>
        <div><img src="../../img/serviceImg/detail_29.png" style="width:6.89rem;margin-bottom: .65rem;" /></div>
        <div><img src="../../img/serviceImg/detail_30.png" style="width:6.9rem;margin-bottom: .55rem;" /></div>
        <div><img src="../../img/serviceImg/detail_31.png" style="margin-bottom:.58rem;" /></div>
        <div><img src="../../img/serviceImg/detail_32.png" style="width:6.59rem;margin-bottom: 1.1rem;" /></div>
      </div>
    </article>
    <div class="s_btn"><router-link :to="{path: 'appointmentBuy'}">预约购买</router-link></div>
  </section>
</template>
<script type="text/babel">
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        tips1: false,
        title: '玻璃保障服务'
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      document.body.scrollTop = document.documentElement.scrollTop = 0;
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
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_SERVICEDETAIL});
      }
    }
  }
</script>