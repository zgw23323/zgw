<template>
	<transition name="page">
		<section class="ck-doc vipCard">
			<div class="vipCard-pic">
				<img :src="imgBaseUrl+vipData">
			</div>
			<!--<div class="vipCard-des">
				<h2>会员卡使用特权</h2>
				<p>会员用户享受折后价格会员用户享受折后价格
会员用户享受折后价格</p>
			</div>-->
		</section>
	</transition>
</template>

<script type="text/babel">
import {setShopsId} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
  export default {
    data() {
      return {
        title: '我的会员卡',
        imgBaseUrl: window.imgBaseUrl,
        apiVipUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxUserCardInfo?orgCode='+window.orgCode,
        vipData: {}
      }
    },
    watch: {
      '$route.path': 'markInit'   //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchVipData();
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_VIPCARD});
      },
      fetchVipData() {
        //获取会员卡数据请求方法
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiVipUrl+'&shopsid='+window.shopsid)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.vipData = data.var;
              /*if(that.vipData.description == '') {
                that.vipData.description = '<p>暂无描述</p>'
              }*/
            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>
