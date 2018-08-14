<template>
  <transition name="page">
    <section class="ck-doc register-result">
      <div class="register-result-box">
        <div class="register-result-icon"></div>
        <div class="register-result-des">申请成功</div>
        
        <div v-if="news == 'recharge'">
          <div class="register-result-mes">
            充值成功后，会员<br>身份即可生效噢！
          </div>
          <div class="register-result-btn" @click="goodsGo('recharge')">去充值</div>
        </div>
        <div v-else>
          <div class="register-result-mes">
            购买商品成功后，会员<br>身份即可生效噢!
          </div>
          <div class="register-result-btn" @click="goodsGo('goods')">去购买</div>
        </div>
        
      </div>
    </section>
  </transition>
</template>

<script>
  import { mapState } from 'vuex';
  import {setShopsId} from '../utils/assist';
  import {LOGCODE} from '../utils/logCode';
  export default {
    data() {
      return {
        title: '会员申请成功'
      }
    },
    computed: {
      ...mapState([
        'news'
      ])
    },
    watch: {
      '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_REGISTERRESULT});
      },
      goodsGo(link) {
        //跳去商品列表页面
        this.$router.replace('/' + link);
      }
    }
  }
</script>
