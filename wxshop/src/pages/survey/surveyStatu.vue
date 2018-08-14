<template>
  <section class="surveyStatu surveyIndex" v-if="suveryStatu != 2">
    <!-- 首次领取成功——3 -->
    <div class="survey_statu1_bg" v-if="suveryStatu == 3">
      <div class="top">
        <p class="top-p1">{{userName}}</p>
        <p class="top-p2">恭喜您获得10元代金卷!</p>
      </div>
      <div>
        <div class="ka-top">
          <p class="ka-top-left"><span class="ka-top-symbol">￥</span>10</p>
          <div class="ka-top-right">
            <p class="ka-top-right-title">售后调研代金卷</p>
            <p class="ka-top-right-li">1.满100元使用</p>
            <p class="ka-top-right-li">2.有效期1年内有效</p>
            <p class="ka-top-right-li">3.不能跨店使用</p>
            <p class="ka-top-right-li">4.仅用于售后维修保养</p>
          </div>
        </div>
        
        <div class="ka-bottom">
          <p class="ka-bottom-p">您的代金券已经在你的会员系统，到店后告知服务顾问即可使用！</p>
          <!-- <img src="../../img/surveyImg/membershipCard.png" class="ka-bottom-img" readonly> -->
        </div>
      </div>
      <img class="logo" src="../../img/surveyImg/logo.png">
    </div>
    <!-- 重复参与1 -->
    <div class="survey_index_bg" v-else>

      <div class="top-2">
        <p class="top-p3">您已参与过本次活动!</p>
      </div>
      <div class="start">
        <p class="start-p1">本次所得代金券已在您的会员系统</p>
        <p class="start-p2">到店后告知服务顾问即可使用</p>
        <!-- <button class="start-button" @click="goTo('serviceHome')">会员中心</button> -->
      </div>
      
      <div class="bottom">
        <p class="bottom-p">活动说明 :</p>
        <p class="bottom-p">1.本活动每人仅限参与一次;</p>
        <p class="bottom-p">2.同一台车/手机号/微信号，均视为同一人;</p>
        <p class="bottom-p">3.本次活动最终解释权归友道会所有.</p>
      </div>
      <img class="logo" src="../../img/surveyImg/logo.png">
    </div>
  </section>
  <section class="surveyStatu2" v-else>
     <!-- 非会员参与调研，未领取 -->
    <div class="suvery_statu2_bg">
      <div class="tick">
        <p class="tick-p">您尚未成为友道会会员!</p>
      </div>

      <a class="prompt-a" href="https://mp.weixin.qq.com/s/G6vBs7ehDI9anrqDM8ApxQ">什么是友道会?</a>
      <p class="prompt-p">请联系店内工作人员在会员系统中为您开通会员开通后，回到公众号点击原链接，即可继续领奖</p>
    </div>
  </section>
</div>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,getStore,setStore,getSessionStore,removeSessionStore} from '../../utils/assist';
import {codeKeyValue} from '../../utils/errorCode';
import {LOGCODE} from '../../utils/logCode';
import ajaxModel from '../../utils/ajaxModel';
export default { 
  data() {
    return {
        title: '',
        suveryStatu:'',
        userName:''
    }
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
  },
   
  methods: {
    markInit() {
      //初始化页面标题
      this.suveryStatu = this.$route.query.statu;
      if(this.suveryStatu == 3) {
        this.title = '恭喜您';
        let that = this;
        ajaxModel.wxUserPersonalCenter(this,(result)=>{
          that.userName = result.userName;
        });
      }else if (this.suveryStatu == 1){
        this.title = '重复参与';
      }else{
        this.title = '诚邀入会';
      }
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
      this.$store.dispatch('log', {account:LOGCODE.SURVEY.PAGE_SURVEYSTATU});
    },
    goTo(pRouter){
      //跳到
      this.$router.push({path: '/'+pRouter});
    }
  }
}
</script>
<style lang="less">
  @import "../../styles/common/survey.less";
</style>