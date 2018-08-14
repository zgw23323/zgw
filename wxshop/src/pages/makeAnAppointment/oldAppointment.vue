<template>
    <section class="c-doc appointmentService">
      <div v-if="tips2" class="windows">
        <!-- <div class="img_test">
            <img class="portrait" :src="userOpt.logopath" v-if="userOpt.logopath">
            <img class="portrait" src="../../img/nav_icon_me_nor@3x.png" v-else>
            <p class="name" v-if="userOpt.userStatus !=3">{{userOpt.userName}}</p>
            <p class="name" v-else><router-link :to="{path: 'login'}">登录</router-link></p>
            <p class="phone">{{userOpt.phone}}<i class="icon_phone" style="display:none;">&nbsp;</i></p>
        </div> -->

        <div>
          <div class="photo">
            <img class="portrait" :src="userOpt.logopath" v-if="userOpt.logopath" />
            <img class="portrait" src="../../img/nav_icon_me_nor@3x.png" v-else />
          </div>

          <div class="datum" v-if="userOpt.userStatus !=3">
            <p class="datum-name" >{{userOpt.userName}}</p>
            <p class="datum-phone">{{userOpt.phone}}<i class="icon_phone" style="display:none;">&nbsp;</i></p>
          </div>

          <div class="datum" v-else>
            <p class="datum-logIn">点击登陆</p>
          </div>
        </div>

        <div style="display: flex;" @click="goTo('myCar')">
          <!-- <p class="carNumber">粤B 45623</p> -->
            <input class="car" v-model="defaultCar.licensePlate" placeholder="预约车牌号" readonly>
            <img style="width: 0.19rem; height: 0.33rem; margin: 0.60rem 0rem 0rem -0.50rem;" src="../../img/appointment/arrow-r.png" />
        </div>
            <!-- <div @click="goTo('appointmentStore')">
              <div class="type" :class="{hide: appointmentObj.orgId == ''}">
                <img style="width: 0.32rem;height: 0.22rem;" src="../../img/appointment/store.png">
                <span>门店</span>
              </div>
              <div style="display: flex;">
                <img class="ico" :class="{hide: !appointmentObj.orgId == ''}" style="width: 0.32rem;height: 0.22rem;top: 0.35rem;" src="../../img/appointment/store.png">
                <input class="box" :class="{font: !appointmentObj.orgId == ''}" v-model="orgName" placeholder="请选择预约门店">
                <img class="arrow-r" src="../../img/appointment/arrow-r.png">
              </div>
            </div> -->
            <div class="type"  @click="goTo('appointmentStore')">
              <div class="type-top" :class="{hide: appointmentObj.orgId == ''}">
                 <img class="type-top-car" src="../../img/appointment/car.png" /><p class="type-top-title">门店</p>
              </div>
              <div class="type-bottom" v-if="appointmentObj.orgId == ''">
                 <div class="type-bottom-car"></div>
                 <input type="text" class="type-bottom-text" v-model="orgName" placeholder="请选择预约门店" readonly>
              </div>
              <div class="type-bottom" v-else>
                 <input type="text" class="type-bottom-text-else" v-model="orgName" placeholder="请选择预约门店" readonly>
              </div>
            </div>
            <!-- <div @click="openTips">
              <div class="type" :class="{hide: Service == ''}">
                <img style="width: 0.30rem;height: 0.30rem;" src="../../img/appointment/service.png">
                <span>类别</span>
              </div>
              <div style="display: flex;">
                <img class="ico" :class="{hide: !Service == ''}" src="../../img/appointment/service.png">
                <input class="box" :class="{font: !Service == ''}" disabled readonly="readonly" v-model="Service" placeholder="请选择预约类别">
                <img class="arrow-r" src="../../img/appointment/arrow-r.png">
              </div>
            </div> -->

            <div class="type" @click="openTips">
              <div class="type-top" :class="{hide: Service == ''}">
                 <img class="type-top-type" src="../../img/appointment/service.png" /><p class="type-top-title">类别</p>
              </div>
              <div class="type-bottom" v-if="Service == ''">
                 <div class="type-bottom-type"></div>
                 <input type="text" class="type-bottom-text" disabled readonly="readonly" v-model="Service" placeholder="请选择预约类别" readonly>
              </div>
              <div class="type-bottom" v-else>
                 <input type="text" class="type-bottom-text-else" v-model="Service" placeholder="请选择预约类别" readonly>
              </div>
            </div>

            <!-- <div @click="goToPersonnel('appointmentTime')">
              <div class="type" :class="{hide: appointmentObj.appointTimeStr  == ''}">
                <img style="width: 0.30rem;height: 0.30rem;" src="../../img/appointment/time.png">
                <span>时间</span>
              </div>

              <div style="display: flex;">
                <img class="ico" :class="{hide: !appointmentObj.appointTimeStr == ''}" src="../../img/appointment/time.png">
                <input class="box" :class="{font: !appointmentObj.appointTimeStr == ''}" v-model="appointmentObj.appointTimeStr" placeholder="请选择预约时间">
                <img class="arrow-r" src="../../img/appointment/arrow-r.png">
              </div>
            </div> -->

            <div class="type" @click="goToPersonnel('appointmentTime')">
              <div class="type-top" :class="{hide: appointmentObj.appointTimeStr == ''}">
                 <img class="type-top-date" src="../../img/appointment/time.png" /><p class="type-top-title">时间</p>
              </div>
              <div class="type-bottom" v-if="appointmentObj.appointTimeStr == ''">
                 <div class="type-bottom-date"></div>
                 <input type="text" class="type-bottom-text" v-model="appointmentObj.appointTimeStr" placeholder="请选择预约时间" readonly>
              </div>
              <div class="type-bottom" v-else>
                 <input type="text" class="type-bottom-text-else" v-model="appointmentObj.appointTimeStr" placeholder="请选择预约时间" readonly>
              </div>
            </div>

            <!-- <div @click="goToPersonnel('buttPersonnel')">
              <div class="type" :class="{hide: peopleName == ''}">
                <img style="width: 0.30rem;height: 0.30rem;" src="../../img/appointment/people.png">
                <span>工作人员</span>
              </div>

              <div style="display: flex;">
                <img class="ico" :class="{hide: !peopleName == ''}" src="../../img/appointment/people.png">
                <input class="box" :class="{font: !peopleName == ''}" v-model="peopleName" placeholder="请选择接待工作人员">
                <img class="arrow-r" src="../../img/appointment/arrow-r.png">
              </div>
            </div> -->

            <div class="type" @click="goToPersonnel('buttPersonnel')">
              <div class="type-top" :class="{hide: peopleName == ''}">
                 <img class="type-top-people" src="../../img/appointment/people.png" /><p class="type-top-title">工作人员</p>
              </div>
              <div class="type-bottom" v-if="peopleName == ''">
                 <div class="type-bottom-people"></div>
                 <input type="text" class="type-bottom-text" v-model="peopleName" placeholder="请选择接待工作人员" readonly>
              </div>
              <div class="type-bottom" v-else>
                 <input type="text" class="type-bottom-text-else" v-model="peopleName" placeholder="请选择接待工作人员" readonly>
              </div>
            </div>

            <!-- <button class="button">我要预约</button> -->
            <div class="b_btn no_btn" v-if="appointmentObj.orgId == ''|| appointmentObj.licensePlate == ''|| appointmentObj.appointTimeStr == ''|| (appointmentObj.isMaintain ==2 && appointmentObj.isRepair==2 && appointmentObj.isSpray==2) || appointmentObj.appointTimeStr == ''">我要预约</div>
            <div class="b_btn" @click="submit" v-else>我要预约</div>
      </div>
      <!-- 预约类别弹窗 -->
       <yd-popup v-model="tips" v-if="tips" position="bottom" class="popup" height="42.5%">
          <!-- <div class="popup" v-if="tips"> -->
            <!-- 确认取消按钮 -->
            <div class="head i_flex_between">
              <div @click="closeTip(-1)">取消</div>
              <div @click="closeTip(1)">确定</div>
            </div>

            <!-- 预约类别选择 -->
            <div class="tip_content">
              <div class="i_flex_between" v-for="item,index in serviceType">
                <p>{{item.serviceName}}</p><div :class="{'no_gou':item.value==2,'cur_gou':item.value==1}" @click="mainBtn(item)">&nbsp;</div>
              </div>
            </div>
          <!-- </div> -->
      </yd-popup>

       <!-- 预约成功弹窗 -->
       <yd-popup v-model="tips1" v-if="tips1" position="center" class="appointment_succes" width="84%" height="42.5%">
          <div class="t_content">
            <div class="t_top">您的预约信息已提交成功，工作人员会尽快与您联系，请保持电话畅通状态。</div>
            <div class="t_s" @click="succesBtn"><span>确定</span></div>
          </div>
      </yd-popup>
    </section>
</template>

<script>
import {Store,setSessionStore,getSessionStore,removeSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '预约申请',
        Service:"",   //预约类别
        peopleName:"",
        apiWxAddAppoint: window.baseUrl+window.appName+'/shopapi/func/appoint/wxAddAppoint?shopsid='+window.shopsid,
        imgBaseUrl: window.imgBaseUrl,
        tips: false,
        tips2: false,
        tips1:false,
        isDefault:true,
        orgName:'',
        appointmentObj: {
          orgId:'', //预约门店ID
          licensePlate:'', //车牌号
          isMaintain:2, //是否预约保养
          isRepair:2, //是否预约维修
          isSpray:2, //是否预约钣喷
          appointTimeStr:'', //预约时间
          saId:'' //工作人员ID
        },
        serviceType:[
          {serviceName:'汽车保养',key:'isMaintain',value:2},
          {serviceName:'汽车维修',key:'isRepair',value:2},
          {serviceName:'钣喷',key:'isSpray',value:2}
        ],
        defaultCar: {licensePlate:''},
        carList:[],//我的车辆信息
        userOpt: {},//用户信息
        appointmentMessage:{}                       //接待工作人员
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.$dialog.loading.open('加载数据中...');
      ajaxModel.wxUserPersonalCenter(this,this.fetchHomeData,'appointmentService')
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_APPOINTMENTSERVICE});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        var orgOpt = getSessionStore('appointment_store',true);
        if(typeof(orgOpt) != 'boolean') {
          this.orgName = orgOpt.orgShortName;
          this.appointmentObj.orgId = orgOpt.id;
        }
        var strTime = getSessionStore('appointment_time');
        if(typeof(strTime) != 'boolean') {
          this.appointmentObj.appointTimeStr = strTime;
        }
        var appointment = getSessionStore('appointment_message',true);
        if(typeof(appointment) != 'boolean') {
          var that = this;
          this.serviceType.forEach((item)=>{
            item.value = appointment[item.key];
            console.log('that.appointmentObj',item.key,appointment[item.key]);
            if(appointment[item.key] == 1){
              that.Service = item.serviceName;
              that.appointmentObj[item.key] =1;
            }
          })
        }
         //获取缓存中的工作人员
        var _personel = getSessionStore('appointment_buttPersonel',true);
        if(typeof(_personel) != 'boolean') {
          this.peopleName = _personel.saName;
          this.appointmentObj.saId = _personel.id;
        }
      },
      openTips(){
        this.tips = true;
      },
      closeTip(pN){
        if(pN == 1) {
          let _ser = this.serviceType.filter((data)=>{
            return data.value ==1;
          })
          if(_ser.length){
            this.Service = '';
            for(var i=0;i<_ser.length;i++){
              this.appointmentObj[_ser[i].key] = 1;
              this.Service = this.Service==''? _ser[i].serviceName : this.Service+'、'+ _ser[i].serviceName;
            }
          }else{
            this.Service = '';
            this.$dialog.notify({mes: '请选择服务类型', icon: 'error', timeout: 800});
            return false
          }
        }
        this.tips = false;
      },
      goTo(pRouter){
        setSessionStore('appointment_message',this.appointmentObj);
        setSessionStore('myCarList_info',this.carList);
        this.$router.push({
          path: '/'+pRouter,
          query: {}
        });
      },
      //跳去选择工作人员列表页面
      goToPersonnel(str){
        if(this.appointmentObj.orgId == ''){
          this.$dialog.notify({mes: '请选择预约门店', icon: 'error', timeout: 800});
            return;
        }
        if(str == 'buttPersonnel'){
          if(this.appointmentObj.appointTimeStr == ''){
            this.$dialog.notify({mes: '请选择预约日期', icon: 'error', timeout: 800});
              return;
          }
        }
        this.goTo(str);
      },
      fetchHomeData(result){
        let that = this;
        this.userOpt = result;
        //判断logo地址是微信头像还是系统图片
        if(this.userOpt.logopath&&this.userOpt.logopath.split('http').length==1){
          this.userOpt.logopath = that.imgBaseUrl + this.userOpt.logopath;
        }
          ajaxModel.wxMyCars(this,(result) => {
            this.$dialog.loading.close();
            that.tips2 = true;
            var _isDefault = result.filter((data)=>{
              return data.isDefault == 1;
            })
            that.carList = result;
            var carOpt = getSessionStore('checkCar_info',true);
            if(typeof(carOpt) != 'boolean'){
              that.defaultCar = carOpt;
            }else{
              if(_isDefault.length){
                that.defaultCar = _isDefault[0];
              }else if(that.carList.length){
                that.defaultCar = that.carList[0];
              }
            }
            that.appointmentObj.licensePlate = that.defaultCar.licensePlate;
          })
      },
      //预约类型点击
      mainBtn(pItem){
        pItem.value = pItem.value == 2 ? 1 : 2;
      },
      succesBtn() {
        this.tips1 = false;
        this.$router.push({path: '/appointmentMessage'});
      },
      //预约提交
      submit() {
        //提交数据
        var that = this;
        if(this.appointmentObj.licensePlate==''){
          this.$dialog.notify({mes: '请选择预约车辆', icon: 'error', timeout: 800});
          return;
        }
        if(this.appointmentObj.orgId == "") {
          this.$dialog.notify({mes: '输选择预约门店', icon: 'error', timeout: 800});
          return;
        }
        if(this.appointmentObj.isMaintain == 2 && this.appointmentObj.isRepair == 2 && this.appointmentObj.isSpray == 2) {
          this.$dialog.notify({mes: '请选择预约类型', icon: 'error', timeout: 800});
          return;
        }
        if(this.appointmentObj.appointTimeStr == "") {
          this.$dialog.notify({mes: '输选择预约时间', icon: 'error', timeout: 800});
          return;
        }
        ajaxModel.wxAddAppoint(this,() => {
          removeSessionStore('appointment_store');
          removeSessionStore('appointment_time');
          removeSessionStore('appointment_message');
          removeSessionStore('appointment_buttPersonel');
          console.log('haha->success');
          that.tips1 = true;
        },this.appointmentObj)
      }
    }
  }
</script>
<style lang="less">
  @import "../../styles/common/chokui.less";
  @import "../../styles/common/style.less";
</style>