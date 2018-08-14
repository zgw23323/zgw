<template>
    <section class="ck-doc appointment">
      <div id="input_item">
        <div class="item">
            <span>手机号：</span>
            <input type="text" v-model="optApply.phone" @focus="handleFocus" v-on:blur="changeInput" maxlength="11" placeholder="请输入手机号">
        </div>
        <div class="item">
            <span>姓名：</span>
            <input type="text" v-model="optApply.customerName" @focus="handleFocus" v-on:blur="changeInput" placeholder="请输入姓名">
        </div>
        
        <div class="item">
            <span>车牌号：</span>
            <input type="text" v-model="optApply.licensePlate" @focus="handleFocus" v-on:blur="changeInput" placeholder="请输入预约车牌号">
        </div>
      </div>
      <div id="flex_bottom">
        <div :class="{'s_btn dis':isKeyUp,'s_btn':!isKeyUp}" @click="submit" >提交</div>
      </div>

      <!--选择法律领域-->
      <yd-popup v-model="tips1" position="center" class="ser-legalfield" width="88%">
          <section class="appointent_tip">
              <div class="tip_top">
                <div>恭喜您，预约购买成功！</div>
                <div class="tip_close_icon" @click="closeTip">&nbsp;</div>
              </div>
              <div class="tip_content">
                  <p class="mess_p1">您需要到4S店审核您车辆玻璃情况， 即可购买成功噢。</p>
                  <p class="mess_p2">本服务支持到店支付</p>
                  <div class="tip_btn" @click="closeTip">确定</div>
              </div>
          </section>
      </yd-popup>
    </section>
</template>

<script>
import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
import {getStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '填写预约购买信息',
        phoneStatus: '',
        uNameStatus:'',
        tips1: false,
        isKeyUp:false,
        btnShow:true,
        winHeight:0,
        apiSubmit : window.baseUrl+window.appName+'/shopapi/func/insurance/wxApplyRecordManage?shopsid='+window.shopsid+"&type=add",
        apiGetInfoUrl:window.baseUrl+window.appName+'/shopapi/func/insurance/wxMatchOpenInfo?shopsid='+window.shopsid,//获取用户信息
        optApply: {
          customerName: "",
          phone:"",
          licensePlate: "",
          recordType:'1'
          
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      document.title = this.title;
      var appointData = getSessionStore('appointmentBuyInfo',true);
        if(typeof(appointData) !='boolean') {
          this.optApply = appointData;
        }
      // this.setScrollHeight();
      this.winHeight = document.documentElement.clientHeight;
      this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_APPOINTMENTBUY});
    },
    watch: {
      'optApply.phone'() { //监听当前输入手机号并实时校验
        if(ydFunLib.telInvalid(this.optApply.phone)) {
          this.fetchUserData();
          this.phoneStatus = "success";
        }else {
          this.phoneStatus = "error";
        }
      },
      'optApply.customerName'() {
        if(!ydFunLib.uNameValid(this.optApply.customerName,true)) {  //监听当前输入用户名并实时校验
          this.uNameStatus = "success";
        }else {
          this.uNameStatus = "error";
        }
      }
    },
    methods: {
      closeTip() {
        this.tips1 = false;
        this.$router.back();
      },
       handleFocus(){
        this.isKeyUp = true; 
        var that = this;
        window.onresize = function(){
          var windowInnerHeight = document.documentElement.clientHeight;
          console.log('windowInnerHeight1',windowInnerHeight);
          if(windowInnerHeight>=that.winHeight){
            console.log('windowInnerHeight2',windowInnerHeight);
            that.isKeyUp = false;
            that.btnShow = true;
          }else {
            that.isKeyUp = true;
            that.btnShow = false;
             console.log('windowInnerHeight3',windowInnerHeight,that.btnShow);
          }
        }
      },
      changeInput(){
        var that = this;
        if(ydFunLib.getCurNavigator().isiOS){
          that.isKeyUp = false;
        }
      },
       //设置滑动区域的最大高度
      setScrollHeight(){
        var bodyHeight = document.body.offsetHeight;
        var item1 = document.getElementById("input_item").offsetHeight;
        var item2 = document.getElementById("flex_bottom").offsetHeight;
        var _height = bodyHeight-item1-item2;
        document.getElementById('input_item').style='margin-bottom:'+_height+'px';
      },
      //获取用户信息
      fetchUserData(){
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiGetInfoUrl,{"phone":that.optApply.phone})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(data.var.isData == 1){
                for(var i=0;i<data.var.matchList.length;i++){
                  var item = data.var.matchList[i];
                  if(item.licensePlate != '' && item.frameNumber !=''){
                    that.optApply.licensePlate = item.licensePlate;
                    that.optApply.customerName = item.name;
                    break;
                  }
                }
              }
            }
            setSessionStore('appointmentBuyInfo',this.optApply);//缓存预约购买信息
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
      submit() {
        var that = this;
        //提交数据
        if(this.uNameStatus =="success" && this.phoneStatus == "success") {
          this.$dialog.loading.open('保存中...');
          setSessionStore('appointmentBuyInfo',this.optApply);//缓存预约购买信息
          this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.BTN_PAGE_APPOINTMENTBUY_SAVE});
          this.$http.post(this.apiSubmit,this.optApply)
            .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_APPOINTMENTBUY_SAVE_OK});
              that.tips1 = true;
            }else {
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_APPOINTMENTBUY_SAVE_FAIL});
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });

        }else {
          if(this.optApply.customerName==''){
            this.$dialog.notify({mes: '请输入姓名', icon: 'error', timeout: 800});
            return;
          }else{
            if(this.uNameStatus != "success") {
              this.$dialog.notify({mes: '姓名输入不正确', icon: 'error', timeout: 800});
              return;
            }
          }
          if(this.optApply.phone==''){
            this.$dialog.notify({mes: '请输入手机号', icon: 'error', timeout: 800});
            return;
          }else{
            if(this.phoneStatus != "success") {
              this.$dialog.notify({mes: '输入手机号错误', icon: 'error', timeout: 800});
              return;
            }
          }
        }
      }
    }
  }
</script>
