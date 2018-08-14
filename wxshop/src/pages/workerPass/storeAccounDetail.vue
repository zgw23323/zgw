<template>
  <transition name="page">
    <section class="storeAccounDetail">
       
       <article class="content">
          <article class="head">
            <p class="p1">玻璃保障服务</p>
            <p>购买详情</p>
          </article>
          <div class="item">
            <div><i class="item_icon1">&nbsp;</i><span>车主信息</span></div>
            <ul>
              <li>车主姓名：{{accountOpt.customerName}}</li>
              <li>车主手机：{{accountOpt.phone}}</li>
              <li>车牌号：{{accountOpt.licensePlate}}</li>
              <li>车架号：{{accountOpt.frameNumber}}</li>
            </ul>
          </div>
          <div class="item">
            <div><i class="item_icon2">&nbsp;</i><span>服务信息</span></div>
            <ul>
              <li v-if="accountOpt.insuranceType==1">购买服务：玻璃保障服务</li>
              <li>购买价格：<span>￥{{accountOpt.insurancePrice}}</span></li>
              <li>服务时间：{{accountOpt.startDateString}} 至 {{accountOpt.endDateString}}</li>
              <li>所属门店：{{accountOpt.orgName}}</li>
              <li>业务员：{{accountOpt.salesmanName}}</li>
              <li>保险公司系统计费报价单</li>
              <li>
                <img :src="accountOpt.costFileUrl" @click="$refs.child.openTip(accountOpt.costFileUrl)" v-if="!isWx"/>
                <img :src="accountOpt.costFileUrl" @click="imgBtn(accountOpt.costFileUrl)" v-if="isWx"/>
              </li>
              <li v-if="accountOpt.agreementFileUrl">服务协议</li>
              <li v-if="accountOpt.agreementFileUrl">
                <img :src="accountOpt.agreementFileUrl" @click="$refs.child.openTip(accountOpt.agreementFileUrl)" v-if="!isWx"/>
                <img :src="accountOpt.agreementFileUrl" @click="imgBtn(accountOpt.agreementFileUrl)" v-if="isWx"/>
              </li>
            </ul>
          </div>
          <div class="s_btn" v-if="accountOpt.financeStatus==1 || accountOpt.financeStatus==null ||accountOpt.financeStatus==0" @click="submitBtn">确认收款</div>
       </article>

      <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
      <yd-imgBig-tip ref="child"></yd-imgBig-tip>
    </section>
  </transition>
</template>
<script type="text/babel">
import { mapState } from 'vuex';
import {removeSessionStore,setSessionStore,getSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
export default {

  data() {
    return {
      title: '详情信息',
      accountOpt:{},
      isWx:true,
      tips4:false,
      imgId:'',
      imgBaseUrl: window.imgBaseUrl,
      storeSubUrl: window.baseUrl+window.appName+"/shopapi/func/insurance/wxFinanceConfim",
      storeApiUrl: window.baseUrl+window.appName+'/shopapi/func/insurance/wxGetOpenInfo?openId='+window.openId//门店列表
    }
  },
  watch: {
    '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据
      document.title = this.title;
      this.fetchMain();
      this.isWx = ydFunLib.curNavigator();
      this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_STOREACCOUNTDETAIL});
  },
   
  methods: {
    submitBtn() {
      let that = this;
      var _openRecordId = window.location.href.split('openRecordId=')[1].split('&')[0];
      that.$http.post(that.storeSubUrl,{'openRecordId':_openRecordId})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              var openOpt = getSessionStore('openRecordId_mess',true);
              if(typeof(openOpt) != 'boolean') {
                openOpt.financeStatus = 2;
                openOpt.isSubmit = true;
                setSessionStore('openRecordId_mess',openOpt);
              }
              this.$dialog.toast({mes: '确认收款成功', icon: 'success', timeout: 2000 ,callback: () => {
                  that.$router.back();
                  that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_STOREACCOUNTDETAIL_SUBMIT_OK});
                }});
            }else {
              that.$dialog.notify({mes: '确认收款失败，请稍后再试！', icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_STOREACCOUNTDETAIL_SUBMIT_FAIL});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
    },
    imgBtn(imgUrl){
      ydFunLib.wxBigImg(imgUrl);
    },
    fetchMain(){
      let that = this;
      var _openRecordId = window.location.href.split('openRecordId=')[1].split('&')[0];
      that.$http.post(that.storeApiUrl,{'openRecordId':_openRecordId})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.accountOpt = data.var;
              if(that.accountOpt.costFileUrl){
                that.accountOpt.costFileUrl = that.accountOpt.costFileUrl.split('http://').length>1?that.accountOpt.costFileUrl:that.imgBaseUrl+that.accountOpt.costFileUrl;
              }
              if(that.accountOpt.agreementFileUrl){
                that.accountOpt.agreementFileUrl = that.accountOpt.agreementFileUrl.split('http://').length>1?that.accountOpt.agreementFileUrl:that.imgBaseUrl+that.accountOpt.agreementFileUrl;
              }
              that.imgId = that.accountOpt.costFileUrl.split('?')[0].split('/shop/downLoad/showLogo/')[1];
              console.log('that.accountOpt',that.accountOpt);
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
    }
  }
}
</script>
