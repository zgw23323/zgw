<template>
  <section class="ck-doc appointmentMessage">
    <div class="prompt">
      <img src="../../img/appointment/success.png" />
      <p>预约已申请</p>
      <span>预约信息已申请，工作人员会尽快与您联系，请保持电话畅通。</span>
    </div>

    <div class="data">

      <div style="display: flex; align-items:center;">
        <img src="../../img/appointment/data.png" />
        <span>预约信息</span>
      </div>

      <div>
        <p>姓名：{{baseInfo.userName}}</p>
        <p>手机号：{{baseInfo.userPhone}}</p>
        <p>车牌号：{{baseInfo.licensePlate}}</p>
        <p>预约门店：{{baseInfo.orgName}}</p>
        <p>预约时间：{{baseInfo.appointTime}}</p>
        <p v-if="baseInfo.saName">接待工作人员：{{baseInfo.saName}}</p>
        <p v-else>接待工作人员：系统自动分配</p>
      </div>
    </div>
  </section>
</template>

<script>
import {getStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '预约成功',
        baseInfo:{
          userName:'',
          userPhone:'',
          licensePlate:'',
          appointTime:'',
          orgName:'',
          saName:''
        },
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      var _urlOpt = ydFunLib.getUrlValue();
      ajaxModel.wxUserAppoint(this,this.fecthData);
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_APPOINTMENTSERVICE});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
      },
      fecthData(result){
        this.baseInfo = result.baseInfo;
      }
    }
  }
</script>