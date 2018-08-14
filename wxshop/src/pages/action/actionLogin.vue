<template>
  <transition name="page">
     <section class="infoPushApply actionLogin">
        <div id="information-bd">
          <div class='item'>                         
            <span slot="left">手机号：</span>
            <input slot="right" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.mobilePhone" type="text" placeholder="请输入您的手机号" maxlength="11">          
          </div>
          
          <div class='item i_flex_betwent'>
            <div style="width: 65%;">
              <span slot="left">验证码：</span>
              <input placeholder="请输入验证码" style="width: 50%;" @focus="handleFocus" v-on:blur="changeInput" type="text" maxlength="6" v-model='optApply.vcode'/>
            </div>                       
            <yd-sendcode slot="right" style="padding:0 .1rem;" v-model="start1" @click.native="sendCode" second="60"></yd-sendcode>
          </div>
          <div class='item'>                        
            <span slot="left"><span style="margin-right:.3rem;">姓</span>名：</span>
            <input slot="right" type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.userName" placeholder="请输入您的姓名" maxlength="12">           
          </div>  
          <div class='item'>                        
            <span slot="left">车牌号：</span>
            <input slot="right" type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.platnumber" placeholder="请输入您的车牌号" maxlength="12">           
          </div>  
          <div class='item'>                        
            <span slot="left">车架号：</span>
            <input slot="right" type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.vin" placeholder="请输入您的车架号" :maxlength="vinlen">           
          </div>        
      </div>
      <div id="information-ft" :class="{'information-ft dis':isKeyUp,'information-ft':!isKeyUp}" @click="submit('submit')">登录</div>
       
    </section>
  </transition>
</template>

<script type="text/javascript">
  
</script>

<script>
  import { mapState } from 'vuex';
  import {setShopsId,removeSessionStore,setSessionStore,getSessionStore} from '../../utils/assist';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
     data() {
        return {
          title: '登录',
          apiUpdateUserInfoUrl: window.baseUrl+window.appName+'/shopapi/func/user/getUserAndRegiste',
          optApply: {//发起支付前 要验证的参数
            vcode:"",
            orgCode: window.orgCode,
            mobilePhone:"",
            vin:"",
            platnumber:"",
            openId: window.openId,
            userName:"",
            method:''
          },
          optCode: {
            phone:''
          },
          vcode:'',
          codeFlag:true,
          phoneStatus: '',
          vCodeStatus: '',
          start1:false,
          isKeyUp:false,
          apiSendUrl: window.baseUrl+window.appName+'/loginapi/appdevice/login?orgCode='+window.orgCode+'&openId='+window.openId,//获取短信验证码
          apiVinUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxGetVinLength?orgCode='+window.orgCode,
          vinlen: 17
        }
    },
    watch: {
      '$route.path': 'markInit', //监听当前路由变化（辅助初始化）
      'optApply.mobilePhone'() { //监听当前输入手机号并实时校验
        if(ydFunLib.telInvalid(this.optApply.mobilePhone)) {
          this.phoneStatus = "success";
          if(this.vCodeStatus && this.codeFlag){
            this.submit('get');
          }
        }else {
          this.phoneStatus = "error";
        }
      },
      'optApply.vcode'() { //监听当前输入验证码并实时校验
        if(ydFunLib.vCodeInvalid(this.optApply.vcode)) {
          console.log('code');
          this.vCodeStatus = "success";
          this.optCode.vcode = this.optApply.vcode;
          this.submit('get');
        }else {
          this.vCodeStatus = "error";
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchVinLength();
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        //获取缓存个人数据
        this.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_ACTIONLOGIN});
      },
      handleFocus(){
        var that = this;
        this.isKeyUp = true; 
        this.winHeight = document.documentElement.clientHeight;
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
      //发送短信
      sendCode(){ 
        //获取验证码请求方法
          if(this.phoneStatus != "success") {
            this.$dialog.notify({mes: '请输入手机号', icon: 'error', timeout: 1000});
            return;
          }else {
            this.$dialog.loading.open('发送中...');
            this.optCode.phone = this.optApply.mobilePhone;
            this.optCode.idCode = '';
            this.optCode.type = 'get';
            // this.$store.dispatch('log', {account:LOGCODE.HOME.BTN_SENDCODE_GET});
            this.$http.post(this.apiSendUrl,this.optCode)
              .then((response) => {
                const data = response.data;
                this.$dialog.loading.close();
                if(data.code == "S_OK") {
                  this.start1 = true;
                  this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_SENDCODE_GET_OK});
                }else {
                  this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_SENDCODE_GET_FAIL});
                  this.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
                }
              },(response) => {
                setTimeout(()=>{
                  this.$dialog.loading.close();
                  this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
                },300);
              });
          }
      },
      fetchVinLength() {
        //获取车架号最低位
        let that = this;
        that.$http.post(that.apiVinUrl+'&shopsid='+window.shopsid)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.vinlen = data.var.vinLength ? data.var.vinLength : that.vinlen;
            }
          });
      },
      submit(str) {
        console.log('submit');
        //提交数据
        let that = this;
        if(ydFunLib.isEmptyValid(that.optApply.mobilePhone)) {
            that.$dialog.notify({mes: '电话号码不能为空', icon: 'error', timeout: 1200});
            return;
        }else{
          if(!ydFunLib.telInvalid(that.optApply.mobilePhone)) {
            that.$dialog.notify({mes: '手机号输入有误', icon: 'error', timeout: 1200});
            return;
          }
        }
        if(this.vCodeStatus != "success") {
          that.$dialog.notify({mes: '请输入6位验证码', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.userName) && str=='submit') {
            that.$dialog.notify({mes: '名字不能为空', icon: 'error', timeout: 1200});
            return;
        }else{
          if(ydFunLib.strlenValid(that.optApply.userName) > 12 && str=='submit') {
            that.$dialog.notify({mes: '输入名字过长', icon: 'error', timeout: 1200});
            return;
          }
        }
        //车架号校验
        if(ydFunLib.isEmptyValid(that.optApply.platnumber) && str=='submit') {
          that.$dialog.notify({mes: '车牌号不能为空', icon: 'error', timeout: 1200});
          return;
        }
        //车架号校验
        if(!ydFunLib.isEmptyValid(that.optApply.vin) && str=='submit') {
          if(ydFunLib.strlenValid(that.optApply.vin) > 17) {
            that.$dialog.notify({mes: '输入车架号不能超过17位', icon: 'error', timeout: 1200});
            return;
          }else if(ydFunLib.strlenValid(that.optApply.vin) < that.vinlen) {
            that.$dialog.notify({mes: '输入车架号不能低于'+that.vinlen+'位', icon: 'error', timeout: 1200});
            return;
          }else if(!ydFunLib.isNumberLetters(that.optApply.vin)) {
            that.$dialog.notify({mes: '请输入您的车架号格式不正确', icon: 'error', timeout: 1200});
            return;
          }
        }else if(str=='submit'){
          that.$dialog.notify({mes: '车架号不能为空', icon: 'error', timeout: 1200});
          return;
        }
        that.optApply.method = str;
        //开始提交
        if(str=='get'){
          that.$dialog.loading.open('加载中...');
        }else{
          that.$dialog.loading.open('保存中...');
        }
        
        that.$store.dispatch('log', {account:LOGCODE.ACTION.BTN_PAGE_ACTIONLOGIN_APPLY});
        that.$http.post(that.apiUpdateUserInfoUrl,this.optApply)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();
            if(data.code == "S_OK") {
              that.$store.dispatch('log', {account:LOGCODE.ACTION.MESS_PAGE_ACTIONLOGIN_APPLY_OK});
              if(str=='get'){
                  that.optApply.vin = data.var.userinfo.vin;
                  that.optApply.platnumber = data.var.userinfo.platnumber;
                  that.optApply.userName = data.var.userinfo.userName;
              }else{
                that.$dialog.notify({mes: '登录成功', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.back();
                }});
              }
            }else if(data.code=="USER_REGISTER_QUERY_5006") {
              // that.$store.dispatch('log', {account:LOGCODE.ACTION.PAGE_ACTIONLOGIN_MESS_APPLY_FAIL});
              that.$dialog.notify({mes: '车辆信息已注册', icon: 'error', timeout: 1200});
            }else{
              that.$dialog.notify({mes: data.msg, icon: 'error', timeout: 1200});
            }
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