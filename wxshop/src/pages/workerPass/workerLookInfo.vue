<template>
  <transition name="page">
    <section class="ck-doc workerLookInfo">
      <article class="header"><img src="../../img/serviceImg/woeker_logo.png" /></article> 
      <article class="content">
        <div class="w_item1">
          <p class="p1">用户预约购买提醒</p>
          <p class="p2">{{personData.documentDate}}</p>
          <p class="p2 w_margin">亲，车主{{personData.customerName}}预约购买了玻璃保障服务~~</p>
        </div>
        <div class="w_item2">
          <ul>
            <li>预约信息</li>
            <li>车牌号码：<span>{{personData.licensePlate}}</span></li>
            <li>车主电话：<span>{{personData.phone}}</span></li>
            <li>操作时间：<span>{{personData.createTime}}</span></li>
            <li>预约时间：<span>以客户到店时间为定</span></li>
            <li>预约项目：<span>玻璃保障服务</span></li>
            <li>预约门店：<span>{{personData.orgName}}</span></li>
          </ul>
          <p>请及时与客户联系！</p>
        </div>
        <div class="w_btn"><a :href="'tel:'+personData.phone">拨打客户电话</a></div>
      </article>
    </section>
  </transition>
</template>

<script type="text/babel">
import {setShopsId,getSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
export default {
    data() {
        return {
          title:'服务权限',
          apiUrl: window.baseUrl+window.appName+'/shopapi/func/insurance/wxQueryApplyInfo?openId='+ window.openId,
          personData: {
            phone: '',
            licensePlate: '',
            createTime: '',
            documentDate: '',
            customerName: '',
            orgName: ''
          }
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
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        //获取缓存个人数据
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_WORKERLOOKINFO});
      },
      fetchInfoData: function() {
        //获取用户预约记录数据
        let that = this;
        var _id = window.location.href.split('id=')[1].split('&')[0];
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl,{'id':_id})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              var varOpt = data.var.applyInfo;
              that.personData.phone = varOpt.phone;
              that.personData.licensePlate = varOpt.licensePlate;
              that.personData.customerName = varOpt.customerName;
              that.personData.orgName = varOpt.orgName;
              var _curDate = varOpt.createTime.split(' ');
              var _date = _curDate[0].split('-');
              that.personData.createTime = _date[0]+'.'+_date[1]+'.'+_date[2]+' '+_curDate[1].substring(0,5);
              this.personData.documentDate = parseInt(_date[1])+'月'+parseInt(_date[2])+'日';
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
