<template>
    <section class="c-doc buttPersonnel">
      <ul>
        <li :class="{'cur i_flex':curBtnId==item.id,'i_flex':curBtnId!=item.id}" v-for="item in buttList" @click="saBtn(item)">
          <div class="left"><img src="../../img/appointment/carLogo.png" /></div>
          <div class="right">
            <p class="p1"><span>{{item.saName}}</span><span class="default" v-if="item.workStatus==2">休假</span></p>
            <p class="p2">售后主管</p>
          </div>
        </li>
      </ul>
      <div class="d_fixed" @click="saBtn()"><div class="fixed_btn add">不选择，由系统分配</div></div>
    </section>
</template>

<script>
import {getStore,getSessionStore,setSessionStore,removeSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '对接工作人员',
        buttList:[],
        curBtnId:'',//当前选择的工作人员
        opt: {}
      }
    },
    mounted() {
      this.markInit();
      console.log('this.opt',this.opt);
      
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_BUTTPERSONNEL});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        var appointmentOpt = getSessionStore('appointment_message',true);
        if(typeof(appointmentOpt) != 'boolean') {
          this.opt = appointmentOpt;
        }
        //获取缓存中的工作人员
        var _personel = getSessionStore('appointment_buttPersonel',true);
        if(typeof(_personel) != 'boolean') {
          this.curBtnId = _personel.id;
        }
        ajaxModel.querySaListForDate(this,this.fecthSa,{orgId:this.opt.orgId,dayDate:this.opt.appointTimeStr.split(' ')[0]});
      },
      fecthSa(result){
        this.buttList = result;
      },
      saBtn(p){
        if(p){
          setSessionStore('appointment_buttPersonel',p);
        }else{
          removeSessionStore('appointment_buttPersonel');
        }
        this.$router.push({path: '/appointmentService'});
      }
    }
  }
</script>
