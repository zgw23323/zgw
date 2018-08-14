<template>

 <!-- <transition name="page">-->
    <section class="ck-doc login">

      <div class="login-wrap">
        <div class="login-bd">
          <div class="ck-input" :data-status="phoneStatus">
            <input v-model="uPhone" type="text" placeholder="请输入手机号码" maxlength="11"/>
            <i class="iconfont i-success">&#xe72e;</i>
          </div>
          <div class="ck-input" :data-status="vCodeStatus">
            <input v-model="vCode" type="text" placeholder="请输入验证码" maxlength="6" >
            <yd-sendcode slot="right" v-model="start1" @click.native="sendCode" second="60"></yd-sendcode>
          </div>
        </div>
        <div class="login-ft">
          <div class="ck-btn">
            <div class="btn" @click="submit">登录</div>
          </div>
        </div>
      </div>

      <div v-if="logoShow" class="login-ft-logo ck-logo"></div>

    </section>
<!--  </transition>-->

</template>

<script>
import {removeStore,getStore,setStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
import {ydFunLib} from '../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '登录',
        uPhone: '',
        vCode: '',
        logoShow: true,
        phoneStatus: '',
        vCodeStatus: '',
        start1: false,
        apiLogin : window.baseUrl+window.appName+'/loginapi/appdevice/login?orgCode='+window.orgCode,
        optsLogin: {
          corpId: window.corpId,
          source: "wechat",
          phone: "",
          idCode:"",
          type: "",
          orgCode: window.orgCode,
        },
        apiCartUrl: window.baseUrl+window.appName+'/shopapi/func/product/addToBasket?orgCode='+window.orgCode,
        optsCart: {}
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      document.title = this.title;
      //获取缓存数据
      let _saveCart = getStore("_saveCart",true);
      if(typeof(_saveCart) === 'boolean') {
        //console.log("Cart is Empty!");
      }else {
        this.optsCart.prodIds = _saveCart;
      }
      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_MSGBOUND});
    },
    watch: {
      uPhone() { //监听当前输入手机号并实时校验
        if(ydFunLib.telInvalid(this.uPhone)) {
          this.phoneStatus = "success";
        }else {
          this.phoneStatus = "error";
        }
      },
      vCode() { //监听当前输入验证码并实时校验
        if(ydFunLib.vCodeInvalid(this.vCode)) {
          this.vCodeStatus = "success";
        }else {
          this.vCodeStatus = "error";
        }
      }
    },
    methods: {
      sendCode() {
        //获取验证码请求方法
        if(this.phoneStatus != "success") {
          this.$dialog.notify({mes: '请输入手机号', icon: 'error', timeout: 1000});
          return;
        }else {
          this.$dialog.loading.open('发送中...');

          this.optsLogin.idCode = '';
          this.optsLogin.phone = this.uPhone;
          this.optsLogin.type = 'get';
          this.$store.dispatch('log', {account:LOGCODE.HOME.BTN_SENDCODE_GET});
          this.$http.post(this.apiLogin,this.optsLogin)
            .then((response) => {
              const data = response.data;
              this.$dialog.loading.close();
              if(data.code == "S_OK") {
                this.start1 = true;
                this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_SENDCODE_GET_OK});
              }else {
                this.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
                this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_SENDCODE_GET_FAIL});
              }
            },(response) => {
              setTimeout(()=>{
                this.$dialog.loading.close();
                this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
              },300);
            });
        }
      },
      saveCartList() {
        //保存购物车列表（异步提交）
        let that = this;
        that.$http.post(that.apiCartUrl+'&shopsid='+window.shopsid,that.optsCart)
          .then((response) => {
          const data = response.data;
          if(data.code == "S_OK" || data.code == "NO_DATA") {
            //清除购物车未登录时候商品详情数组缓存
            removeStore('_cartDetailsData');
          }else if(data.code == "NO_LOGIN" || data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
            //清除购物车未登录时候商品详情数组缓存
            removeStore('_cartDetailsData');
          }
        });
      },
      submit() {
        //提交数据
        if(this.vCodeStatus =="success" && this.phoneStatus == "success") {

          this.optsLogin.idCode = this.vCode;
          this.optsLogin.phone = this.uPhone;
          this.optsLogin.type = 'valid';

          this.$dialog.loading.open('登录中...');
          this.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_MSGBOUND_SAVE});
          this.$http.post(this.apiLogin,this.optsLogin)
            .then((response) => {
            const data = response.data;

            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              this.$dialog.toast({mes: '登录成功', icon: 'success', timeout: 1000 ,callback: () => {
                this.$router.back();
                //缓存数据
                window.shopsid = data.var.sid;
                setStore("_shopsid", window.shopsid)
                //提交购物车缓存列表数据
                this.saveCartList();
                this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_MSGBOUND_SAVE_OK});
              }});
            }else {
              this.$dialog.notify({mes: '验证码错误', icon: 'error', timeout: 800});
              this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_MSGBOUND_SAVE_FAIL});
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });

        }else {
          if(this.vCodeStatus != "success") {
            this.$dialog.notify({mes: '请输入6位验证码', icon: 'error', timeout: 800});
            return;
          }
          if(this.phoneStatus != "success") {
            this.$dialog.notify({mes: '输入手机号错误', icon: 'error', timeout: 800});
            return;
          }
        }
      }
    }
  }
</script>
