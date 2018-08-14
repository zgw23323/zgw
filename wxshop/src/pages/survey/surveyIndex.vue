<template>
  <section class="surveyIndex">
    <div class="survey_index_bg">
      <div class="top">
        <p class="top-p1">恭喜您!</p>
        <p class="top-p2">获得友道会本次有奖调研参与资格!</p>
      </div>
      <div class="start">
        <p class="start-p1">感谢您在百忙中的参与!</p>
        <p class="start-p2">本次调研主要针对广州“四开四停”政策，为能在新政策实施之时，更好的服务大家，特进行本次调研。您的每一条作答，都会将我们服务更进一步。</p>
        <button class="start-button" @click="startSurvey">开始调研</button>
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
</div>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
import {codeKeyValue} from '../../utils/errorCode';
import {LOGCODE} from '../../utils/logCode';
import ajaxModel from '../../utils/ajaxModel';
export default { 
  data() {
    return {
        title: '代步车调研',
        userInfoPhone:''
    }
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
    let that = this;
    // ajaxModel.wxUserPersonalCenter(this,(result)=>{
    //   that.userInfoPhone = result.phone;
    //   setSessionStore('userInfoPhone',result.phone);
    //   setSessionStore('userInfoName',result.userName);
    // });
  },
   
  methods: {
     
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
       

      this.$store.dispatch('log', {account:LOGCODE.SURVEY.PAGE_SURVEYINDEX});
    },
    startSurvey() {
      let that = this;
      this.$store.dispatch('log', {account:LOGCODE.SURVEY.BTN_PAGE_SURVEYINDEX_APPLY});
      ajaxModel.wxIsJoinSurvey(this,(result)=>{
        if(!result.isJoin){
          that.$router.push({
            path: '/surveyStore',
            query: {
              from: 'surveyIndex',
              isJoin: result.isJoin,
              receiveStatus: result.receiveStatus
            }
          });
        }else if(result.receiveStatus == 2) {
          that.$router.push({
            path: '/surveyLogin'
          });
        }else {
          that.$router.push({
            path: '/surveyStatu',
            query:{
              statu:'1'
            }
          });
        }
      })
    }
  }
}
</script>
<style lang="less">
  @import "../../styles/common/survey.less";
</style>
