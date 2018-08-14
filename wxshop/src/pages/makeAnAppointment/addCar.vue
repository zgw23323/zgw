<template>
    <section class="c-doc addCar">
        <div class="branch" @click="goTo">
          <p class="title">车系:</p>
          <input type="text" class="text" :class="{crude: !opt.carsName == ''}" v-model="opt.carsName" placeholder="请选择车系">
          <img class="arrow-r" src="../../img/appointment/arrow-r.png" />
        </div>

        <div class="branch">
          <p class="title">车牌号:</p>
          <p class="title" style="width: 1.40rem;" @click="$refs.child.openTip()">{{chaeckArea}}<img class="arrow-d" src="../../img/appointment/arrow-d.png"></p>
          <input type="text" class="text" style="width: 3.15rem;" :class="{crude: !licensePlate == ''}" v-model="licensePlate" placeholder="请输入车牌号">
        </div>
        <div class="branch" v-show="!datetime2">
          <p class="title">购车时间:</p>
          <yd-datetime type="month" v-model="opt.buyCarDate"  ref="datetimeBtn" class="text" slot="right" ></yd-datetime>
          <img class="arrow-r" src="../../img/appointment/arrow-r.png" />
        </div>
        <div class="branch" v-if="datetime2" >
          <p class="title">购车时间:</p>
          <input type="text" class="text" @focus="tiemFocus(1)" :class="{'crude': !opt.buyCarDate == ''}" placeholder="请选择购车时间">
          <img class="arrow-r" src="../../img/appointment/arrow-r.png" />
        </div>
        <div class="branch">
          <p class="title">里程:</p>
          <input type="text" class="text" style="width: 3.55rem;" :class="{crude: !opt.carMileage == ''}" v-model="opt.carMileage" placeholder="请输入当前里程">
          <p class="title" style="width: 1.00rem;">公里</p>
        </div>

        <div class="branch">
          <p class="title">车架号:</p>
          <input type="text" class="text" :class="{'crude': !opt.frameNumber == ''}" :maxlength="vin" v-model="opt.frameNumber" placeholder="请输入车架号（可选填）">
        </div>
        
        <!-- 设置了 “车系、车牌、购车时间、行驶里程” 四样都填写才可 保存 -->
        <div class="d_fixed">
          <div @click="submit" class="fixed_btn add" v-if="opt.carsId != ''&& opt.licensePlate != '' && opt.buyCarDate != '' && opt.carMileage != ''">保存</div>
          <div class="fixed_btn add no_btn" v-else>保存</div>
        </div>
        <!-- 车牌归属地选择 -->
        <yd-cut-area ref="child" v-on:child-tell-me-something='listenToMyBoy'></yd-cut-area>
    </section>
</template>

<script>
import {getStore,setSessionStore,getSessionStore,removeSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '添加车辆',
        chaeckArea:"粤",   //车牌归属地
        datetime2: true,
        licensePlate:'',
        vin:17,
        vinStatus:'',
        opt: {
          carsId: '',
          carsName:'',
          licensePlate:'',
          buyCarDate:'',
          carMileage:'',//里程数
          frameNumber:''
        }
         
      }
    },
     watch: {
      '$route.path': 'markInit',  //监听当前路由变化（辅助初始化）
      'opt.frameNumber'() {
        if(!this.vinValid(this.opt.frameNumber)) {  //监听当前输入车架号并实时校验
          if(ydFunLib.strlenValid(this.opt.frameNumber) < this.vin) {
            this.vinStatus = "error";
          }else if(ydFunLib.strlenValid(this.opt.frameNumber) > 17) {
            this.vinStatus = "error";
          }else if(!ydFunLib.isNumberLetters(this.opt.frameNumber)) {
            this.vinStatus = "error";
          }else {
            this.vinStatus = "success";
          }
        }else {
          this.vinStatus = "error";
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      document.title = this.title;
      var _curDate = ydFunLib.getCurDate();
      this.markInit();
      // ajaxModel.wxGetVinLength(this,(result)=>{
      //   this.vin = result.vinLength;
      // });
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_ADDCAR});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        var _opt = getSessionStore('addCarSava_info',true);
        if(typeof(_opt) != 'boolean'){
          this.opt = _opt;
          if(_opt.licensePlate){
            var _len = _opt.licensePlate.split(' ');
            this.chaeckArea = _len[0];
            this.licensePlate =_len[1];
          }
        }
        var _optCar = getSessionStore('addCar_info',true);
        if(typeof(_optCar) != 'boolean'){
          this.opt.carsName = _optCar.carsName;
          this.opt.carsId = _optCar.carsId;
        } 
      },
      vinValid(value,allowEmptyString) { //校验车架号方法
        let val= value.replace(/\s/g,"");
        return (val === null) || (val === undefined)
          || (!allowEmptyString ? val === '' : false)
          || (val.length === 0);
      },
      tiemFocus(){
        var that = this;
        that.datetime2 = false;
        that.$refs.datetimeBtn.open();
      },
      listenToMyBoy(str){
        this.chaeckArea = str;
      },
      goTo() {
        this.opt.licensePlate = this.chaeckArea+' '+this.licensePlate;
        setSessionStore('addCarSava_info',this.opt);
        this.$router.push({
          path: '/carList',
          query: {from:'addCar'}
        });
      },
      submit() {
        //提交数据
        var that = this;
        if(this.opt.carsId==''){
          this.$dialog.notify({mes: '请选择车系', icon: 'error', timeout: 800});
          return;
        }
        if(this.licensePlate == "") {
          this.$dialog.notify({mes: '请填写车牌号', icon: 'error', timeout: 800});
          return;
        }else{
          this.opt.licensePlate = this.chaeckArea+' '+this.licensePlate;
        }
        if(this.opt.buyCarDate == "") {
          this.$dialog.notify({mes: '请选择购车时间', icon: 'error', timeout: 800});
          return;
        }else{
          this.opt.buyCarDate = this.opt.buyCarDate+'-01';
        }

        if(this.opt.carMileage == "") {
          this.$dialog.notify({mes: '请填写里程数', icon: 'error', timeout: 800});
          return;
        }
        if(this.opt.frameNumber != "") {
          if(this.vinStatus == 'error'){
            this.$dialog.notify({mes: '请输入正确车架号', icon: 'error', timeout: 800});
            return;
          }
        }
        ajaxModel.wxAddUserCar(this,(result) => {
          removeSessionStore('addCar_info');
          removeSessionStore('addCarSava_info');
          removeSessionStore('myCarList_info');
          this.$dialog.loading.close();
            that.$router.push({
              path: '/myCar'
            });
        },this.opt)
      }
    }
  }
</script>