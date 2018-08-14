<template>
	<section class="ck-doc consumeDetail">
    <article class="head">
        <div class="item">
          <p>我的余额</p>
          <p>￥2340</p>
        </div>
        <div class="item">
          <div class="line_l">&nbsp;</div>
          <div>
            <p>我的积分</p>
            <p>2340</p>
          </div>
        </div>
    </article>
    <article class="content">
      <ul>
        <li class="li_1">
          <div>消费项目</div>
          <div>消费金额</div>
          <div>消费积分</div>
        </li>
        <li class="li_2">
          <div>
            <p>2/10</p>
            <p class="p2">商城充值</p>
          </div>
          <div class="p3">+300</div>
          <div class="p4">-</div>
        </li>
        <li class="li_2">
          <div>
            <p>2/10</p>
            <p class="p2">商城充值</p>
          </div>
          <div class="p3">-300</div>
          <div class="p3">+40000</div>
        </li>
      </ul>
    </article>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        title:'消费详情',
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/serve/rightsListDriveDot?update=true',
        infoDataList:[],
        showType:''
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchInfoData();
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
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_SERVICECITY});
        document.body.scrollTop = document.documentElement.scrollTop = 0;
      },
      
      fetchInfoData: function() {
        //获取首页数据
        let that = this;
        // that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>