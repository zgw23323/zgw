<template>
	<section class="ck-doc">
		<div class="Expressinfo">
      <div class="top">
        <p class="WaybillNum">运单号：{{canshu1}}</p>
        <p class="company">快递公司：{{canshu2}}</p>
      </div>
      <yd-timeline class="info">
        <yd-timeline-item v-for="v,index in expressInfo" :key='index'>
            <p>{{v.context}}</p>
            <p style="margin-top: 10px;">{{v.time}}</p>
        </yd-timeline-item>
      </yd-timeline>
    </div>
	</section>
</template>

<script type="text/babel">
import {setShopsId,getSessionStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
import ajaxModel from '../utils/ajaxModel';
import axios from 'axios';
import {emsCode} from '../utils/emsCode';
export default {
	data() {
		return {
			title: '快递追踪',
      canshu1:"",
      canshu2:"",
      expressInfo:[]
		}
	},
  watch: {
    '$route.path': 'markInit'  //监听当前路由变化 （辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据

    this.markInit();
    let that = this;
    let orderStatus = that.$route.query.orderStatus;
    let expressObj = getSessionStore('ExpressStatu_info',true);
    if(orderStatus != 4 && expressObj.expressNumber) {
      this.canshu1 = expressObj.expressNumber;
      this.query_ExpressInfo(expressObj.expressCompanyName,expressObj.expressNumber);
    } else {
      let item = {context:'发货中',time:expressObj.expressTime};
      this.expressInfo.push(item);
    }
  },
  methods: {
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
      //获取门店信息

      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_EMSSTATUS});
    },
    query_ExpressInfo (code,num) {
      this.$dialog.loading.open('正在加载中');

      var then = this;
      var comCode = [];

      comCode = emsCode.filter(function(data){
        return data.companyCode == code;
      })
      if(comCode.length < 1){
        then.$dialog.loading.close();
        then.$dialog.notify({mes: '查询失败，快递公司不支持。', icon: 'error', timeout: 1200});
        return;
      } else {
        this.canshu2 = comCode[0].companyName;
      }

      axios({method:'post',url:"/wxpay/kuaidi/api?id=af76da55b3a3d725&com=" + code + "&nu=" + num})
      .then(function(res){
        then.$dialog.loading.close();
        if(res.data.message == 'ok') {
          then.expressInfo = res.data.data;
        } else {
          then.$dialog.notify({
            mes: res.data.message,
            icon: 'error',
            timeout: 1200
          });
        }
      })
      .catch(function(err){
        then.$dialog.loading.close();
        then.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
      })
    }
  }
}
</script>
