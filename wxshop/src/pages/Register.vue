<template>
  <transition name="page">
    <section class="ck-doc">
      <div class="register-wrap">
        <div class="register-bd">
          <div class="ck-input-2">
            <i class="i-icon i-phone"></i>
            <input type="text" disabled placeholder="请输入手机号码" v-model="uPhone"/>
          </div>
          <div class="ck-input-2" :data-status="uNameStatus">
            <i class="i-icon i-user"></i>
            <input type="text" placeholder="请输入您的姓名" v-model="optApply.userName" maxlength="20">
          </div>

          <!--获取白名单车辆信息S-->
          <div class="whitelist-wrap" v-if="whiteFlag">
            <label class="title-tips">请选择一个入会车牌</label>
            
            <div class="whitelist-item" v-for="cd in CustomerData" @click="select(cd)">
              <div :class="cd.vinCode == activityData.vinCode ? 'current' : ''">{{cd.lincenseCode}}</div>
            </div>

            <div class="whitelist-item newadd">
              <div @click="newAddBtn">&nbsp;+&nbsp;新增入会车牌</div>
            </div>
          </div>

          <div v-if="!whiteFlag">
            <div class="ck-input-2" :data-status="plateStatus">
              <i class="i-icon i-plate"></i>
              <input type="text" placeholder="请输入您的车牌号码" v-model="optApply.plateNumber" maxlength="12"/>
            </div>
            <div class="ck-input-2" :data-status="vinStatus">
              <i class="i-icon i-cartype"></i>
              <input type="text" :placeholder="'请输入不少于' + vinlen + '位车架号'" v-model="optApply.vin" maxlength="17"/>
            </div>

            <div class="ck-input-2">
              <i class="i-icon i-tjr"></i>          
              <input type="text" @click="selectReferrer" placeholder="请选择入会推荐人" v-model="optApply.referer" maxlength="17"/>           
            </div>
          </div>
        </div>

        <div class="register-ft">
          <div class="ck-btn">
            <div class="btn" @click="submit">免费申请会员</div>
          </div>
        </div>

      </div>
    </section>
  </transition>
</template>

<script type="text/javascript">
  
</script>

<script>
  import { mapState } from 'vuex';
  import {setShopsId,removeSessionStore,setSessionStore,getSessionStore} from '../utils/assist';
  import {LOGCODE} from '../utils/logCode';
  import {ydFunLib} from '../utils/ydFunLib';
  import ajaxModel from '../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '申请会员',
        apiApplyUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxUserRegister?orgCode='+window.orgCode,
        uPhone: '',
        activityData: {'vinCode':""},
        CustomerData:[],
        whiteFlag:false,
        optApply: {
          phone: '',
          userName: '',
          /*licenseNo: '',
           cartypeName : '',*/
          plateNumber: '',
          referer:'',//推荐业务员姓名
          employeeId:'',//业务员id
          employeeName:'',//业务员姓名
          vin: ''
        },
        uNameStatus: '',
        vinStatus: '',
        plateStatus: '',
        apiVinUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxGetVinLength?orgCode='+window.orgCode,
        vinlen: 6
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      ajaxModel.wxGetVinLength(this,this.fetchVinLength);
      // this.fetchVinLength();
    },
    watch: {
      '$route'(){
        
      },
      '$route.path': 'markInit',  //监听当前路由变化（辅助初始化）
      'activityData.vinCode'(){
        this.optApply.plateNumber=this.activityData.lincenseCode;//车牌号
        this.optApply.vin=this.activityData.vinCode;//车架号

        if(this.whiteFlag)
          this.whiteFlag = false;//关闭选择框
      },
      'optApply.userName'() {
        if(!ydFunLib.uNameValid(this.optApply.userName)) {  //监听当前输入用户名并实时校验
          this.uNameStatus = "success";
        }else {
          this.uNameStatus = "error";
        }
      },
      'optApply.vin'() {
        if(!this.vinValid(this.optApply.vin)) {  //监听当前输入车架号并实时校验
          if(ydFunLib.strlenValid(this.optApply.vin) < this.vinlen) {
            this.vinStatus = "error";
          }else if(ydFunLib.strlenValid(this.optApply.vin) > 17) {
            this.vinStatus = "error";
          }else if(!ydFunLib.isNumberLetters(this.optApply.vin)) {
            this.vinStatus = "error";
          }else {
            this.vinStatus = "success";
          }
        }else {
          this.vinStatus = "error";
        }
      },
      'optApply.plateNumber' () {
        //车牌校验
        /*if(/^[\u4e00-\u9fa5]{1}[A-Z]{1}[A-Z_0-9]{5}$/.test(this.optApply.plateNumber) === false) {
          this.plateStatus = "error";
        }else {
          this.plateStatus = "success";
        }*/
        if (!ydFunLib.isEmptyValid(this.optApply.plateNumber)) {
          this.plateStatus = "success";
        }else {
          this.plateStatus = "error";
        }

      }
    },
    computed: {
      ...mapState([
        'headerTitle', 'news','cardInfo'
      ])
    },
    methods: {
      markInit() {

        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        //获取选择的推荐业务员id
        let select_referrer = sessionStorage.getItem("_select_referrer");
        if (select_referrer != null && select_referrer != '' && select_referrer != undefined) {
          var referObj = JSON.parse(select_referrer);
          this.optApply.referer = referObj.realName;
          this.optApply.employeeId = referObj.id;
          this.optApply.employeeName = referObj.realName;
        }

        this.revertRegInfo();

        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_REGISTER});

      },
      clearStoreReg() {//清理掉临时记录
        removeSessionStore("_store_reg_info");
        removeSessionStore("_select_referrer");
        removeSessionStore("_select_referrer_index");
      },
      revertRegInfo() {//从sessionStorage中恢复用户填写的信息
        let store_reg_fromsession = sessionStorage.getItem("_store_reg_info");
        if (store_reg_fromsession != null && store_reg_fromsession != '' && store_reg_fromsession != undefined) {
          var storeRegObj = JSON.parse(store_reg_fromsession);
          this.optApply.userName = storeRegObj.userName;
          this.optApply.plateNumber = storeRegObj.plateNumber;
          this.optApply.vin = storeRegObj.vin;
        }else 
          return false;        
        return true;
      },
      storeRegisterInfo() {
        //保存用户填写的注册信息
        setSessionStore("_store_reg_info",this.optApply);
      },      
      selectReferrer() {

        this.storeRegisterInfo();
        //跳去地址
        this.$router.push({
          path: '/referrer',
          query: {
            mark: 'change'
          }
        });
      },
      select(acData) {
        this.activityData = acData;
      },
      newAddBtn() {
        //console.log(this);
        this.whiteFlag = false;//手动录入车架号、车牌
        this.optApply.plateNumber='';
        this.optApply.vin='';
      },
      vinValid(value,allowEmptyString) { //校验车架号方法
        let val= value.replace(/\s/g,"");
        return (val === null) || (val === undefined)
          || (!allowEmptyString ? val === '' : false)
          || (val.length === 0);
      },
      applyGo() {
        //跳去申请会员页面
        var query = this.$route.query.f;        
        /*if (query != undefined && query != null) {
          this.$router.push('/' + query);
        }else {
          this.$router.push('/registerResult');
        }*/
        this.$store.commit('UPDATE_NEWS', query );
        this.$router.push('/registerResult');
      },
      fetchVinLength(result) {
        //获取车架号最低位
        let that = this;
          that.vinlen = result.vinLength;
          that.uPhone = result.phone ;
          that.optApply.phone = result.phone;
          
          var isStoreReg = this.revertRegInfo();
          console.log("isStoreReg=", isStoreReg);
          if (!isStoreReg) {
            //根据手机号获取白名单车辆信息
            var cardataArr = result.carDatas;            
            /*cardataArr = [{"vinCode": "LWVCAF755FA004333", "lincenseCode": "粤UBF666", "customerName": "测试1", "mobile1":15801740169},{"vinCode": "LWVCAF755FA004444", "lincenseCode": "粤UBF777", "customerName": "测试2", "mobile1":15801740169}];*/
            
            if (cardataArr != undefined && cardataArr != null && cardataArr.length > 0) {
              that.CustomerData = cardataArr;
              if (cardataArr.length == 1) {
                that.optApply.plateNumber=that.CustomerData[0].lincenseCode;
                that.optApply.vin=this.CustomerData[0].vinCode;
                that.whiteFlag = false;
              }else {
                that.whiteFlag = true;
              }              
            }else {
              that.whiteFlag = false;
            }
          }                        
      },
      submit() {
        //提交数据
        var that = this;
        //校验输入的姓名
        if(that.uNameStatus !="success") {
          that.$dialog.notify({mes: '请输入您的姓名', icon: 'error', timeout: 1200});
          return
        }
       
        if (that.plateStatus != "success") {
          that.$dialog.notify({mes: '请输入车牌号', icon: 'error', timeout: 1200});
          return
        }
        //车牌号校验
        /*if(/^[\u4e00-\u9fa5]{1}[A-Z]{1}[A-Z_0-9]{5}$/.test(that.optApply.plateNumber) === false) {
          that.$dialog.notify({mes: '车牌号输入有误', icon: 'error', timeout: 1200});
          return
        }*/

        //校验输入的车架号
        if(that.vinStatus != "success") {
          if(ydFunLib.strlenValid(that.optApply.vin) > 17) {
            that.$dialog.notify({mes: '输入车架号不能超过17位数', icon: 'error', timeout: 1200});
          }else if(ydFunLib.strlenValid(that.optApply.vin) < that.vinlen) {
            that.$dialog.notify({mes: '输入车架号不能低于'+that.vinlen+'位', icon: 'error', timeout: 1200});
          }else if(!ydFunLib.isNumberLetters(that.optApply.vin)) {
            that.$dialog.notify({mes: '请输入您的车架号格式不正确', icon: 'error', timeout: 1200});
          }else {
            that.$dialog.notify({mes: '请输入您的车架号', icon: 'error', timeout: 1200});
          }
          return
        }

        that.$dialog.loading.open('提交申请中...');
        that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_REGISTER_SAVE});
        that.$http.post(that.apiApplyUrl+'&shopsid='+window.shopsid,that.optApply)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_REGISTER_SAVE_OK});
              that.$dialog.toast({
                mes: '申请成功',
                timeout: 1000,
                icon: 'success',
                callback: () => {
                  this.clearStoreReg();//清理缓存
                  //当申请成功后，更改vuex中的卡号标题 (显示卡申请中)
                  //this.$store.commit('UPDATE_TITLE', '卡申请中');
                  //{'cardNo':'卡申请中', 'linkTo':'AA'}
                  this.$store.commit('UPDATE_CARDINFO', {'cardNo':'卡申请中', 'linkTo':'/registerResult'});
                  
                  let query = this.$route.query.f;
                  this.$store.commit('UPDATE_NEWS', query );

                  this.$router.replace('/registerResult');
                }
              });
              
            }else if(data.code == "NO_LOGIN" || data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                that.$router.push('/login');
              }});
            }else if(data.code == "USERINFO_V_FAL_0X003") {
              that.$dialog.notify({mes: '车架号位数不足', icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_REGISTER_SAVE_FAIL});
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_REGISTER_SAVE_FAIL});
            }
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);
          },(response) => {
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      }
    }
  }
</script>

