<template>

 <!-- <transition name="page">-->
    <section class="ck-doc login service">

      <div class="login-wrap">
        <div class="login-bd">
          <div class="ck-input" :data-status="phoneStatus">
            <input v-model="uPhone" type="number" placeholder="请输入手机号码" maxlength="11"/>
            <i class="iconfont i-success">&#xe72e;</i>
          </div>
          <div class="ck-input" :data-status="vCodeStatus">
            <input v-model="vCode" type="number" placeholder="请输入验证码" maxlength="6" >
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
      
      <!-- 申请会员提示 -->
      <div class="ck-pop-toast" v-if="toast3">
        <div class="inner">
          <div class="ck-pop">
            <div class="ck-pop-close" @click="toastClose"></div>
            <div class="ck-pop-bd">
              <p class="p1">亲，您不是会噢！</p>
              <div class="s-pop-btn" @click="applyGo">免费申请会员</div>
            </div>
          </div>
        </div>
      </div>
    </section>
<!--  </transition>-->

</template>

<script>
import {getStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
import {ydFunLib} from '../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '登录',
        uPhone: '',
        vCode: '',
        toast3: false,
        logoShow: true,
        phoneStatus: '',
        vCodeStatus: '',
        start1: false,
        apiLogin : window.baseUrl+window.appName+'/loginapi/appdevice/login?orgCode='+window.orgCode+'&openId='+window.openId,
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
      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_LOGIN});
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
            localStorage.removeItem('_cartDetailsData');
          }else if(data.code == "NO_LOGIN" || data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
            //清除购物车未登录时候商品详情数组缓存
            localStorage.removeItem('_cartDetailsData');
          }
        });
      },
      toastClose(){
        this.toast3 = false;
      },
      applyGo() {
        //跳去申请会员
        this.$router.push('/vipApply');
      },
      submit() {
        //提交数据
        var that = this;
        if(this.vCodeStatus =="success" && this.phoneStatus == "success") {

          this.optsLogin.idCode = this.vCode;
          this.optsLogin.phone = this.uPhone;
          this.optsLogin.type = 'valid';

          this.$dialog.loading.open('登录中...');
          this.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_LOGIN_SAVE});
          this.$http.post(this.apiLogin,this.optsLogin)
            .then((response) => {
            const data = response.data;

            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              this.$dialog.toast({mes: '登录成功', icon: 'success', timeout: 1000 ,callback: () => {
                that.$router.back();
                //缓存数据
                if(data.var.userinfo.openId){
                  window.shopsid = data.var.userinfo.openId;
                  window.openId = data.var.userinfo.openId;
                  localStorage.setItem("_shopsid", window.shopsid);
                  localStorage.setItem("_openId", window.openId);
                }

                //从预约跳转到的登录，与普通用户身份时跳到完善个人信息页面
                if(that.$route.query.formType =='appointmentService' && data.var.type!=1){
                  that.$router.push('/perfectInfo');
                }
              }});
              localStorage.setItem('serviceInfoPhone', this.uPhone);
              //提交购物车缓存列表数据
              that.saveCartList();
              that.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_LOGIN_MESS_OK});
            }else {
              this.$dialog.notify({mes: '验证码错误', icon: 'error', timeout: 800});
              this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_LOGIN_FAIL});
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });

        }else {
          if(this.uPhone==''){
            this.$dialog.notify({mes: '手机号不能为空', icon: 'error', timeout: 800});
            return;
          }else if(this.phoneStatus != "success") {
            this.$dialog.notify({mes: '输入手机号错误', icon: 'error', timeout: 800});
            return;
          }
          if(this.vCodeStatus != "success") {
            this.$dialog.notify({mes: '请输入6位验证码', icon: 'error', timeout: 800});
            return;
          }
          
        }
      }
    }
  }
</script>
