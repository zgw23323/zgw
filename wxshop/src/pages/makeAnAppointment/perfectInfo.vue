<template>
    <section class="c-doc perfectInfo addCar">
        <div class="branch">
	      <p class="title">姓名:</p>
	      <input type="text" class="text" v-model="opt.userName" placeholder="请输入您的姓名">
	    </div>
	    <div class="branch">
	      <p class="title">手机号:</p>
	      <input type="text" class="text" disabled readonly="readonly" v-model="phone" placeholder="请输入您的手机号">
	    </div>
	    <div class="branch">
	      <p class="title">车牌号:</p>
	      <p class="title" style="width: 1.35rem;"  @click="$refs.child.openTip()">{{chaeckArea}}<img class="arrow-d" src="../../img/appointment/arrow-d.png" /></p>
	      <input type="text" class="text" style="width: 3.24rem;" v-model="licensePlate" placeholder="请输入车牌号">
	    </div>
	    <div class="branch" @click="goTo">
	      <p class="title">车系:</p>
	      <input type="text" class="text" v-model="opt.carName" placeholder="请选择您的车系">
	      <img class="arrow-r" src="../../img/appointment/arrow-r.png" />
	    </div>
	    <div class="d_fixed" @click="submit"><div class="fixed_btn add">保存</div></div>

      <!-- 车牌归属地选择 -->
      <yd-cut-area ref="child" v-on:child-tell-me-something='listenToMyBoy'></yd-cut-area>
    </section>
</template>

<script>
import {getStore,getSessionStore,setSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';

  export default {
    data() {
      return {
        title: '完善个人信息',
        chaeckArea:"粤",   //车牌归属地
        tips:false,
  	    phone:"",   // 手机号
        licensePlate:'',
        opt: {
          carId: '',
          carName:'',
          licensePlate:'',
          userName:''
        }
      } 
    },
    mounted() {
      //挂载完毕初始化页面数据
      document.title = this.title;
      this.markInit();
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_APPOINTMENTSERVICE});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        var _opt = getSessionStore('addCar_info',true);
        if(typeof(_opt) != 'boolean'){
          this.opt.carName = _opt.carsName;
          this.opt.carId = _opt.carsId;
        }
        var _phone = getStore('serviceInfoPhone', false);
        if(typeof(_phone) != 'boolean'){
          this.phone = _phone;
        }
        var _perfect = getSessionStore('perfect_info',true);
        if(typeof(_perfect) != 'boolean'){
          this.opt = _perfect;
          if(_perfect.licensePlate){
            var _len = _perfect.licensePlate.split(' ');
            this.chaeckArea = _len[0];
            this.licensePlate =_len[1];
          }
        }
      },
      goTo() {
        this.opt.licensePlate = this.chaeckArea+' '+this.licensePlate;
        setSessionStore('perfect_info',this.opt);
        this.$router.push({
          path: '/carList',
          query: {from:'perfectInfo'}
        });
      },
      listenToMyBoy(str){
        this.chaeckArea = str;
        this.tips = !this.tips;
      },
      submit() {
        //提交数据
        var that = this;
        if(this.opt.userName==''){
          this.$dialog.notify({mes: '请填写姓名', icon: 'error', timeout: 800});
          return;
        }
        if(this.licensePlate == "") {
          this.$dialog.notify({mes: '请填写车牌号', icon: 'error', timeout: 800});
          return;
        }else{
          this.opt.licensePlate = this.chaeckArea+' '+this.licensePlate;
        }
        if(this.opt.carsId==''){
          this.$dialog.notify({mes: '请选择车系', icon: 'error', timeout: 800});
          return;
        }
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_LOGIN_BTN_SAVE});
        ajaxModel.wxPerfectUserInfo(this,this.opt);
      }
    }
  }
</script>