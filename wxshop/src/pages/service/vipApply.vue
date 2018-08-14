<template>
  <transition name="page">
    <section class="service vipApply">
         <div class="ser-input-2">
            <i class="i-icon i-user"></i>
            <input type="text" placeholder="请输入您的姓名" v-model="personData.userName" maxlength="20">
          </div>
          <div class="ser-input-2">
            <i class="i-icon i-phone"></i>
            <input type="text" placeholder="请输入手机号码" v-model="personData.phone"/>
          </div>
          <div class="ser-input-2">
            <i class="i-icon i-card"></i>
            <input type="text" placeholder="请输入身份证" maxlength="19" v-model="personData.idNumber"/>
          </div>
          <div class="ser-input-2">
            <i class="i-icon i-cart"></i>
            <input type="text" :placeholder="'请输入不少于' + vinlen + '位车架号'" v-model="personData.vin" maxlength="17"/>
          </div>
          <div class="ser-input-2">
            <i class="i-icon i-plate"></i>
            <input type="text" placeholder="请输入您的车牌号码" v-model="personData.plateNumber" maxlength="12"/>
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
  import {setShopsId,removeSessionStore,setSessionStore,getSessionStore} from '../../utils/assist';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
     data() {
        return {
          title: '免费申请会员',
          uheadDefault: true,
          imgBaseUrl: window.imgBaseUrl,
          apiUpdateUserInfoUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxUserRegister?orgCode='+window.orgCode,
          personData: {
            logopath: '',
            userName: '',
            phone: '',
            idNumber: '',
            cartypeName: '',
            plateNumber: '',
            vin: ''
          },
          uhead: '',
          apiVinUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxGetVinLength?orgCode='+window.orgCode,
          vinlen: 6
        }
    },
    watch: {
      optsAddressData: { //深度监听地址数据
          handler() {
          },
          deep: true
      },
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
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
        
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_VIPAPPLY});
      },
      fetchVinLength() {
        //获取车架号最低位
        let that = this;
        that.$http.post(that.apiVinUrl+'&shopsid='+window.shopsid)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.vinlen = data.var.vinLength;
            }
          });
      },
      submit() {
        //提交数据
        let that = this;

        //名字校验
        if(ydFunLib.isEmptyValid(that.personData.userName)) {
            that.$dialog.notify({mes: '名字不能为空', icon: 'error', timeout: 1200});
            return;
        }else{
          if(ydFunLib.strlenValid(that.personData.userName) > 12) {
            that.$dialog.notify({mes: '输入名字过长', icon: 'error', timeout: 1200});
            return;
          }
        }
        if(ydFunLib.isEmptyValid(that.personData.phone)) {
            that.$dialog.notify({mes: '电话号码不能为空', icon: 'error', timeout: 1200});
            return;
        }else{
          if(!ydFunLib.telInvalid(that.personData.phone)) {
            that.$dialog.notify({mes: '手机号输入有误', icon: 'error', timeout: 1200});
            return;
          }
        }
        //身份证校验
        if(ydFunLib.isEmptyValid(that.personData.idNumber)) {
          that.$dialog.notify({mes: '身份证不能为空', icon: 'error', timeout: 1200});
          return;
        }
        if(/(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/.test(that.personData.idNumber) === false) {
          that.$dialog.notify({mes: '身份证输入有误', icon: 'error', timeout: 1200});
          return;
        }

        //车架号校验
        if(!ydFunLib.isEmptyValid(that.personData.vin)) {
          if(ydFunLib.strlenValid(that.personData.vin) > 17) {
            that.$dialog.notify({mes: '输入车架号不能超过17位', icon: 'error', timeout: 1200});
            return;
          }else if(ydFunLib.strlenValid(that.personData.vin) < that.vinlen) {
            that.$dialog.notify({mes: '输入车架号不能低于'+that.vinlen+'位', icon: 'error', timeout: 1200});
            return;
          }else if(!ydFunLib.isNumberLetters(that.personData.vin)) {
            that.$dialog.notify({mes: '请输入您的车架号格式不正确', icon: 'error', timeout: 1200});
            return;
          }
        }else{
          that.$dialog.notify({mes: '车架号不能为空', icon: 'error', timeout: 1200});
          return;
        }

        //开始提交
        that.$dialog.loading.open('保存中...');

        let optsPersonData = {};

        optsPersonData.userName = that.personData.userName;
        optsPersonData.phone = that.personData.phone;
        optsPersonData.idNumber = that.personData.idNumber;
        optsPersonData.plateNumber = that.personData.plateNumber;
        optsPersonData.vin = that.personData.vin;
        that.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_VIPAPPLY_SAVE});
        that.$http.post(that.apiUpdateUserInfoUrl+'&shopsid='+window.shopsid,optsPersonData)
          .then((response) => {
            const data = response.data;
            setTimeout(()=>{
              that.$dialog.loading.close();
              if(data.code == "S_OK") {
                that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_VIPAPPLR_OK});
                that.$dialog.notify({mes: '保存成功', icon: 'error', timeout: 1200, callback: () => {
                    that.$router.back();
                  }});
              }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                    that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/login');
                    }});
                  }else if(data.code == "NO_LOGIN") {
                    that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                      that.$router.push('/login');
                    }});
                  }else {
          that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
        }
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