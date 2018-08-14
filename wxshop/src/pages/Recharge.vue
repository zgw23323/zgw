<template>
  <transition name="page">
    <section class="ck-doc">

      <div class="recharge-wrap" v-if="!tips1">
        <div class="store_select" v-if="orgCode=='YDH'" @click="selectStore">
          <div class="left">门店</div>
          <div class="right"><span>{{orgOpt.orgShortName}}</span><i class="s_icon">&nbsp;</i></div>
        </div>
        <div :class="{'recharge-hd hasT': orgCode!='YDH','recharge-hd': orgCode=='YDH'}" @click="tipShow">
          <p class="u-balance-title">会员卡<span class="item_c">{{cardSelectNumber}}</span></p>
          <p class="u-balance">余额：<span>￥{{parseFloat(cardSelectMoney).toFixed(2)}}</span></p>
          <div class='recharge_right_nav'></div>
        </div>
        <div v-if="showRefer == true" class="ck-r-input recharge-hd"  @click="selectReferrer">
          <p class="u-balance-title">推荐人<span class="item_c">{{optsRecharge.employeeName}}</span></p>

          <i class="i-icon i-tjr"></i>
          <slot name="right"></slot>
        </div>
        <div v-if="couponList.totalCount>0" class=" recharge-hd recharge_coupon"  @click='tipOpen(3)'>
          <div class="item">
            <p class="u-balance-title">优惠券<span class="ac_coupon_bg" v-if="optsCoupon.couponSelectName != ''">{{optsCoupon.couponSelectName}}</span></p>
          </div>
          <div class="item">
            <span class="coupon_price">{{optsCoupon.couponSelectAmount}}</span>
            <span class='recharge_right_nav'></span>
          </div>
        </div>
        <div class="recharge-bd">
          <h2>选择充值金额</h2>
          <div class="recharge-list">
            <div :key="rc.id" :class="{'current':(rc.id == activityId),'current-selover': (rc.selStock <=0 && rc.selStock != -1)}" v-for="rc in ActivityData.recharges" @click="select(rc.id,rc.payMoney,rc.selStock)"><strong>￥{{rc.payMoney}}</strong><br><span><label v-if="rc.giveMoney > 0">送{{rc.giveMoney}}</label><label v-else>&nbsp;</label></span></div>

          </div>

          <div class="recharge-mark" v-html="mark" ></div>

        </div>

        <div class="recharge-ft">
          <div class="ck-btn"> 
            <div :class="{'btn':!isApply,'btn hasBtn':isApply}" @click="pay">确认充值</div>
          </div>
        </div>

      </div>

      <div class="tipsPage" v-if="tips1">
        <div class="tipsBox">
          <div class="tipStatus" data-type="busy"></div>
          <p>亲，网络出问题啦~~</p>
          <div class="btns" @click="reLoad">重新加载</div>
        </div>
      </div>

      <div class="ck-pop-toast" v-if="toast3">
        <div class="inner">
          <div class="ck-pop">
            <div class="ck-pop-close" @click="toastClose(3)"></div>
            <div class="ck-pop-bd">
              <p class="p1">亲，您还不是会员噢！</p>
              <div class="ck-pop-btn" @click="applyGo">免费申请会员</div>
            </div>
          </div>
        </div>
      </div>

      <!-- 新增会员卡列表弹窗 -->
      <div class='tip_f' v-if="tips2">&nbsp;</div>
      <div class='ck-tipsCar' id="ck-tipsCar" v-if="tips2">
          <section class="absol">
            <div>
              <span @click="cancelCar">取消</span>
            </div>
            <div>
              <span @click="carSubmit">确定</span>
            </div>
          </section>  
          <ul class="shop">
            <li v-for="(rC,index) in cardList" @click='carCheck(rC.cardNumber,rC.balance)'>
              <div class='car_n'>{{rC.cardNumber}}</div>
              <!-- <div class="vip_p">余额：￥{{parseFloat(rC.balance).toFixed(2)}}</div> -->
              <!-- <div :class='{"curr_Vip":(rC.cardNumber == cardNumber)}'>&nbsp;</div> -->
              <div class="vip_p">
                <span class="vip_p">余额：￥{{parseFloat(rC.balance).toFixed(2)}}</span>
                <span :class='{"curr_Vip":(rC.cardNumber == cardNumber)}'>&nbsp;</span>
              </div>
            </li>
          </ul>
      </div>
      <!-- 选择 -->
      <yd-bottom-tip v-if="tips3" :isShow='true' :tipsType='"tips3"'>
        <ul class="od-full-song">
          <li class="item1" v-if="couponList.totalCount>0" v-for="coupon in couponList.userCouponList" @click='couponCheck(coupon.couponId,coupon.couponAmount,coupon.couponName)'><em :class="{'icon_checked c_no':coupon.couponId !=optsCoupon.couponSelectId,'icon_checked c_ing':coupon.couponId ==optsCoupon.couponSelectId}">&nbsp;</em><span>省{{coupon.couponAmount}}元，{{coupon.couponName}}</span></li>
          <li v-else><span>暂无可用优惠券</span></li>
        </ul>
        <div class="ck-fixed-btn" @click='tipClose(3)'>确认选择</div>
      </yd-bottom-tip>     
    </section>
  </transition>

</template>

<script>
import { mapState } from 'vuex';
import {setShopsId,setSessionStore,getSessionStore,removeSessionStore} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
import {ydFunLib} from '../utils/ydFunLib';
import ajaxModel from '../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '会员卡充值',
        tips1: false,
        tips2:false,
        tips3:false,
        toast3: false,
        selected: true,
        selectSel:0,
        orgCode:window.orgCode,
        orgOpt: {
          orgShortName: '友道会',
          id:'0a1a00b55b571a80815b5b5300b20a2d',
          code:window.orgCode
        },
        mark:'',showRefer:true,
        actualPaymentAmount:0,//最终要支付的金额
        optApply: {//发起支付前 要验证的参数
          id: '',
          orderNum:'',
          employeeId:'',//业务员id
          cardNumber:'',//会员卡号ID
          employeeName:''//业务员姓名
        },
        ActivityData: {
          balance: 0
        },
        isWx:true,
        cardList:[],
        activityId: '',
        optsCoupon: {
          couponSelectId:'',
          couponSelectId:'',
          couponSelectName:'',
          couponSelectAmount:'请选择'
        },
        
        aipActivity: window.baseUrl+window.appName+'/shopapi/func/recharge/wxListActivity?orgCode=',
        aipRecharge: window.baseUrl+window.appName+'/shopapi/func/recharge/wxConfirmRecharge?orgCode=',
        //获取充值优惠券列表
        apiProdCouponList:window.baseUrl+window.appName+'/shopapi/func/coupon/userCouponList?orgCode=',
        isApply:true,//提交的时候会根据这个属性来判断是否已经提交过,已经提交过就退出，避免重复提交
        optsRecharge: {
          orderCode: "",
          employeeId:'',//业务员id
          employeeName:''//业务员姓名
        },
        cardNumber:'',//会员卡号
        cardMoney:'0',//会员卡余额
        cardSelectNumber:'',//会员卡号
        cardSelectMoney:'0',//会员卡余额
        payMoney: "",
        urlOpt: {},
        isVip: window.isVip,
        couponList:{
          totalCount: 0,
          totalPage:1,
          currentTime: '',
          userCouponList: []
        },//优惠券列表
        userType: ""
      }
    },
    watch: {
      'activityId'(){
        this.optApply.id = this.activityId;
      },
      '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    //添加钩子，监听路由变化时候跳转问题
    beforeRouteLeave(to, from, next){
     var s_1 = getSessionStore('s_1',false);
      if(typeof(s_1) !='boolean'){
        next();
      }else{
        // setSessionStore('s_1','111');
        if(this.isWx){
          WeixinJSBridge.call('closeWindow');
        }
        next(false);
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      //获取活动数据
      // this.fetchActivityList(this.orgCode);
      //获取优惠券列表
      this.getCouponList();
      this.isWx = ydFunLib.curNavigator();
      removeSessionStore('s_1');
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
      storeRechargeInfo() {
        let selectActId = this.activityId;
        //var storeRegStr = JSON.stringify(storeReg);
        //保存用户选择的充值选项
        sessionStorage.setItem("_store_recharge_info", selectActId);
        //缓存用户勾选满减送信息
        if(this.optsCoupon.couponSelectId != ''){
          sessionStorage.setItem("_coupon_select_info", JSON.stringify(this.optsCoupon));
        }
        //缓存当前选择的门店
        setSessionStore('recharge_selected_store',this.orgOpt.code);
      },      
      selectReferrer() {
        this.storeRechargeInfo();
        setSessionStore('s_1','referrer');
        //跳去地址
        this.$router.push({
          path: '/referrer',
          query: {
            from: 'recharge',
            org: this.orgOpt.code
          }
        });
      },
      selectStore() {
        this.storeRechargeInfo();
        var orgLen = [];
        orgLen.push(this.orgOpt);
        setSessionStore('s_1','Store');
        setSessionStore('information_selected_store',orgLen);
        //跳去地址
        this.$router.push({
          path: '/Store',
          query: {
            from: 'recharge'
          }
        });
      },
      tipOpen(tips){
      this['tips'+tips] = true;
      },
      tipClose(tips){
        this['tips'+tips] = false;
      },
      //选择优惠券
      couponCheck(id,pAmount,pName){
        if(this.optsCoupon.couponSelectId == id){
          this.optsCoupon.couponSelectId = '';
          this.optsCoupon.couponSelectAmount = '请选择';
          this.optsCoupon.couponSelectName = '';
        }else{
          this.optsCoupon.couponSelectId = id;
          this.optsCoupon.couponSelectAmount = '- '+pAmount+'元';
          this.optsCoupon.couponSelectName = pName;
        }
      },
      applyGo() {
        //跳去申请会员
        //this.$router.push('/register');
        //从会员充值跳转到注册页面
        setSessionStore('s_1','register');
        this.$router.push({
          path: '/register',
          query: {
            f:'recharge'
          }
        });
      },
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
         //从缓存中拿到缓存的充值活动列表
        var activity_recharge_data = getSessionStore('activity_recharge_data',true);
        if(typeof(activity_recharge_data) !='boolean') {
          this.ActivityData = activity_recharge_data;
          //会员卡列表
          this.cardList = this.ActivityData.cardList;
        }

        //获取选择的推荐业务员id
        let select_referrer = getSessionStore("recharge_select_referrer",true);        
        if (typeof(select_referrer) != 'boolean') {          
          var referObj = select_referrer;
          this.optsRecharge.employeeId = referObj.id;
          this.optsRecharge.employeeName = referObj.realName;

          this.optApply.employeeId = referObj.id; 
          this.optApply.employeeName = referObj.realName;
        }
        //获取地址上参数
        this.urlOpt = ydFunLib.getUrlValue();
        if(this.urlOpt.refereeId && typeof(select_referrer) == 'boolean'){
          ajaxModel.wxListEmployee(this,this.getEmployee);
        }
        //从sessionStorage中拿到选择的充值活动数据
        let select_activity = sessionStorage.getItem('recharge_selected_activity');
        if(select_activity!= null && select_activity != '' && select_activity != undefined){
            var  activityObj = JSON.parse(select_activity);
            this.activityId = activityObj.activityId;
            this.optApply.id = activityObj.id;
            this.payMoney = activityObj.payMoney;
            this.selectSel = activityObj.selStock;
        }

        //从sessionStorage中拿到选择的优惠券信息
        let select_coupon = sessionStorage.getItem('_coupon_select_info');
        if(select_coupon!= null && select_coupon != '' && select_coupon != undefined){
            var  couponObj = JSON.parse(select_coupon);
            this.optsCoupon = couponObj;
        } 
       
        // this.activityId = sessionStorage.getItem("_store_recharge_info");
        this.revertSelect();

        //获取更改前选择的门店
        var moOrg = getSessionStore('recharge_selected_store');
        //获取选择的门店
        if(this.orgCode == 'YDH'){
          let select_store = getSessionStore('information_selected_store',true);
          if(typeof(select_store) !='boolean'){
            this.orgOpt = ydFunLib.ydExtend(this.orgOpt,select_store[0]);
            if(this.orgOpt.code != this.orgCode){
              this.fetchActivityList(this.orgOpt.code);
              if(typeof(moOrg) !='boolean' && this.orgOpt.code != moOrg){
                this.optsRecharge.employeeId = '';
                this.optsRecharge.employeeName = '';

                this.optApply.employeeId = '';
                this.optApply.employeeName = '';
              }
            }else{
              this.fetchActivityList(this.orgCode);
            }
          }else{
            this.fetchActivityList(this.orgCode);
          }
        }else{
          this.fetchActivityList(this.orgCode);
        }
        
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_RECHARGE});
      },
      getEmployee(arr){
        var _arr = arr.filter(data=>{return this.urlOpt.refereeId==data.id});
        if(_arr.length){
          let item = _arr[0];
          this.optsRecharge.employeeId = item.id;
          this.optsRecharge.employeeName = item.realName;

          this.optApply.employeeId = item.id;
          this.optApply.employeeName = item.realName;
          setSessionStore("recharge_select_referrer",item);
          setSessionStore("recharge_select_referrer_index",item.id);
        }
        
      },
      revertSelect() {
        //从sessionStorage中加载已经选择的卡号
        let selectedCardInfoStr = sessionStorage.getItem("recharge_selected_vipcard");
        let selectedCardInfo = JSON.parse(selectedCardInfoStr);

        if (selectedCardInfo != null && selectedCardInfo != undefined) {          
          this.cardSelectNumber = selectedCardInfo.vipCard;
          this.cardSelectMoney = selectedCardInfo.balance;
        } 
        if(this.cardSelectNumber !='' && this.activityId !=''){
          this.isApply = false;
        };
      },
      toastClose(toast) {
        //关闭弹出提示框
        if(toast == 1) {
          this.toast1 = false;
        }else if(toast == 2) {
          this.toast2 = false;
        }else if(toast == 3) {
          this.toast3 = false;
        }
      },
      select(id,paymoney,selStock) {
        if (selStock <=0 && selStock != -1) {
          this.$dialog.notify({mes: '请选择其他选项.', icon: 'error', timeout: 1200});
          return;
        }
        //选择金额和id
        this.activityId = id;
        this.optApply.id = id;
        this.payMoney = paymoney;
        this.selectSel = selStock;
        this.setSelectActivitySession();
      },
      //设置选择活动信息缓存
      setSelectActivitySession(){
        let selectActivityObj={
          'activityId':this.activityId,
          'id':this.activityId,
          'payMoney':this.payMoney,
          'selectSel':this.selectSel
        };
        sessionStorage.setItem("recharge_selected_activity",JSON.stringify(selectActivityObj));
      },
      fetchActivityList(str) {
        //获取充值页面相关数据
        this.$dialog.loading.open('加载数据中...');
        var _url = this.orgCode == 'YDH'?this.aipActivity+str+'&shopsid='+window.shopsid+'&chooseOrgId='+this.orgOpt.id:this.aipActivity+str+'&shopsid='+window.shopsid
        this.$http.post(_url)
          .then((response) => {
            const data = response.data;
            this.tips1 = false;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              this.ActivityData = data.var;

              this.optsRecharge.orderCode = '';//data.var.orderCode;
              this.optApply.orderNum = '';//data.var.orderCode;
              setSessionStore('activity_recharge_data',data.var);
              var _activity = getSessionStore("recharge_selected_activity",true);
              if(_activity ==''||_activity==null||_activity==undefined){
                  if(data.var.recharges.length != 0) {
                    for(var i=0;i<data.var.recharges.length;i++){
                      if (this.ActivityData.recharges[i].selStock > 0) {
                        this.activityId = this.ActivityData.recharges[i].id;
                        this.payMoney = this.ActivityData.recharges[i].payMoney;
                        this.selectSel = this.ActivityData.recharges[i].selStock;                       
                        this.setSelectActivitySession();
                        break;
                      }
                    }                
                  }
              }else{//此处判断是防止后台停用充值信息，启用别的充值信息，进入到充值页面没有默认选择项
                var len = this.ActivityData.recharges.filter(function(data){
                  return data.id == _activity.id;
                });
                if(len.length==0){
                  for(var i=0;i<this.ActivityData.recharges.length;i++){
                    if (this.ActivityData.recharges[i].selStock > 0) {
                      this.activityId = this.ActivityData.recharges[i].id;
                      this.payMoney = this.ActivityData.recharges[i].payMoney;
                      this.selectSel = this.ActivityData.recharges[i].selStock;                       
                      this.setSelectActivitySession();
                      break;
                    }
                  }    
                }
              }
              this.mark = data.var.mark;
              //会员卡列表
              this.cardList = data.var.cardList;
              if(this.cardList.length>0){
                this.cardNumber=this.cardList[0].cardNumber;
                this.cardMoney=this.cardList[0].balance;
                if(this.cardSelectNumber == null || this.cardSelectNumber == ''){
                  this.selectedVipCardInfo({"vipCard":this.cardNumber, "balance":this.cardMoney});
                }

              }
              this.userType = data.var.userType;
              console.log("usertype: ", this.userType, this.isVip, window.isVip);
              if(window.isVip == false || window.isVip == 0 || this.userType === false) {
                this.toast3 = true;
              }else {
                this.toast3 = false;
              }
              if(this.selectSel <= 0 || this.cardList.length==0){
                this.isApply = true;
              }else{
                this.isApply = false;
              }
            
            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
              setSessionStore('s_1','login');
              this.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                this.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN") {
              setSessionStore('s_1','login');
              this.$dialog.notify({mes: '您还没有登录', icon: 'error', timeout: 1200, callback: () => {
                this.$router.push('/login');
              }});
            }else {
              this.tips1 = true;
              this.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);
          },(response) => {
            this.tips1 = true;
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);
          });
      },
      pay() {
        //判断是否提交过
        if(this.isApply){
          return false;
        }else{
          this.isApply = true;
        }
        //判断是否是会员
        if(this.userType === false) {
          this.toast3 = true;
          this.isApply = false;
          return;
        }else {
          this.toast3 = false;
        }

        if (this.selectSel <= 0) {
          this.$dialog.notify({mes: '请选择一个充值金额.', icon: 'error', timeout: 1200});
          this.isApply = false;
          return;
        }   
        this.checkBeforePay();
      },
      checkBeforePay() {

        let that = this;
        that.optApply.id = that.activityId;
        that.optApply.cardNumber = that.cardSelectNumber;
        if(that.couponSelectId !=''){
          that.optApply.couponId = that.optsCoupon.couponSelectId;
        }
          let wxGetStockRntUrl = window.shopUrl+window.appName+'/shopapi/func/recharge/wxGetStockRnt?orgCode='+this.orgOpt.code;
          that.$dialog.loading.open('请求数据中...');
          that.$http.post(wxGetStockRntUrl+'&shopsid='+window.shopsid+'&type=save',that.optApply)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              this.optsRecharge.orderCode = data.var.orderNum;
              this.optApply.orderNum = data.var.orderNum;
              that.actualPaymentAmount = data.var.actualPaymentAmount;
              sessionStorage.removeItem('_coupon_select_info');
              /**
                1.充值付款前需要做基本参数校验：a.商品数量是否足够。b.参数是否合法
                2.基本参数校验通过后，如果是在本地测试环境，则需要模拟微信支付
              */
              console.log({'orderNumber':this.optApply.orderNum,'payMoney':that.actualPaymentAmount,'activityId':that.activityId});
              if(window.isWXpay == true) {
                  this.getWxPayOpts();
              }else {
                  // console.log(window.shopUrl+window.appName+'/wxpay/finalpay?orderNo='+that.optsRecharge.orderCode+'&money='+that.actualPaymentAmount+'&openId='+window.openId+'&shopsid='+window.shopsid+'&orgCode='+window.orgCode+'&pt=1&acid=' + that.activityId+'&vipCard=' + that.cardSelectNumber);
                  this.optsRecharge.weChatStatus = 1;
                  this.optsRecharge.weChatOrderCode = "test00000000001";
                  this.optsRecharge.returnCode = "get_brand_wcpay_request:ok";
                  this.optsRecharge.returnCodeDesc = "test00000000001";
                  //this.submit();

                  var formData = {
                    func:that.submit,
                    cancel:that.cancelOrder,
                    data:1,
                    forms:{"vipCard": that.cardSelectNumber, "orgCode": this.orgOpt.code, "orderNo": this.optsRecharge.orderCode, "payMoney": that.payMoney, "actId": that.activityId}
                  }
                  //that.optsRecharge.activityId = this.activityId;
                  this.$store.dispatch('simulationPay', formData);
                  this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_RECHARGE_PAY_OK});
              }

              
            }else{
              that.isApply = false;

              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  setSessionStore('s_1','login');
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  setSessionStore('s_1','login');
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
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


      },
      submit() {
        //提交数据
        let that= this;
        that.$dialog.loading.open('提交数据中...');

        that.optsRecharge.activityId = this.activityId;
        that.optsRecharge.vipCard = this.cardSelectNumber;
        that.$http.post(this.aipRecharge+this.orgOpt.code+'&shopsid='+window.shopsid,this.optsRecharge)
          .then((response) => {
            const data = response.data;

        that.$dialog.loading.close();

        that.$store.dispatch('log', {account:'recharge_login_' + data.code});
        
        if(data.code == "S_OK") {
          setSessionStore('s_1','rechargebill');
          that.$dialog.toast({mes: '提交成功', icon: 'success', timeout: 1000 ,callback: () => {
            that.$router.push({
              path: '/rechargebill',
              query: {
                id: data.var
              }
            });
          }});

          sessionStorage.removeItem("recharge_selected_vipcard");
          this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_RECHARGE_MESS_PAY_OK});

        }else{

          if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
            that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
              setSessionStore('s_1','login');
              that.$router.push('/login');
            }});
          }else if(data.code == "NO_LOGIN"){
            that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
              setSessionStore('s_1','login');
              that.$router.push('/login');
            }});
          }else{
            that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
          }
        }
      },(response) => {
        setTimeout(()=> {
          that.$dialog.loading.close();
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
        },300);
      });
    },
    //获取优惠券列表
    getCouponList() {
      let that = this;
        console.log(12);
        that.$http.post(that.apiProdCouponList+this.orgOpt.code+'&shopsid='+ window.shopsid,{curPage:1,pageSize:6,isRecharge:'1',isShop:'',isAvailable:1})
        .then((response) => {
          const data = response.data;
          that.tips1 = false;
          if(data.code == "S_OK") {
            that.couponList = data['var'];
            if(that.couponList.totalCount>0){
              that.couponSelectAmount = '请选择';
            }else{
              that.couponSelectAmount = '无可用';
            }
          }else{
            if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
              that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                setSessionStore('s_1','login');
                that.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN"){
              that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                setSessionStore('s_1','login');
                that.$router.push('/login');
              }});
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    getWxPayOpts() {

        //请求服务器获取微信支付需要相关参数
          let that = this;
          that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_RECHARGE_PAY_PREF});

          // let aipWxpayUrl = window.shopUrl+window.appName+'/wxpay/finalpay?orderNo='+that.optsRecharge.orderCode+'&money='+that.payMoney+'&openId='+window.openId+'&shopsid='+window.shopsid+'&orgCode='+window.orgCode+'&pt=1&acid=' + that.activityId+'&vipCard=' + that.cardSelectNumber;
          let aipWxpayUrl = window.shopUrl+window.appName+'/wxpay/finalpay?orderNo='+that.optsRecharge.orderCode+'&money='+that.actualPaymentAmount+'&openId='+window.openId+'&shopsid='+window.shopsid+'&orgCode='+this.orgOpt.code+'&pt=1&acid=' + that.activityId+'&vipCard=' + that.cardSelectNumber;
          console.log('wxpay=>',aipWxpayUrl);
          that.$dialog.loading.open('请求数据中...');
          that.$http.get(aipWxpayUrl)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();
            if(data.code == "S_OK") {
              //支付成功后修改vuex里边的会员卡卡号为当前支付选择的卡号,供其他页面使用              
              that.$store.commit('UPDATE_VIPCARD', that.cardSelectNumber);
              that.onBridgeReady(data.var.appId, data.var.timeStamp, data.var.nonceStr, data.var.package, data.var.paySign);
            }else {              
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
              that.$store.dispatch('log', {account:'recharge_pay_config_err_' + data.msg});
            }
          },(response) => {
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.$dialog.notify({mes: '服务器请求失败', icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_RECHARGE_PAY_CONFIG_EX});
            }, 300);
          });
      },
      cancelOrder() {
        var that = this;
        that.optApply.id = that.activityId;
        that.optApply.cardNumber = that.cardSelectNumber;
        let wxGetStockRntUrl = window.shopUrl+window.appName+'/shopapi/func/recharge/wxGetStockRnt?orgCode='+this.orgOpt.code;
        that.$dialog.loading.open('取消订单中...');
        that.$http.post(wxGetStockRntUrl+'&shopsid='+window.shopsid+'&type=cannel',that.optApply)
        .then((response) => {
          that.isApply = false;
          const data = response.data;
            if(data.code == "S_OK") {
              //that.$dialog.notify({mes: '已取消', icon: 'error', timeout: 1000});
              //that.$router.go(0);
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_RECHARGE_PAY_CANCEL_OK});
            }
            that.$router.go(0);
            setTimeout(()=>{
              this.$dialog.loading.close();
            },300);
          },(response) => {
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_RECHARGE_PAY_CANCEL_EX});
            },300);
          });
      },
      onBridgeReady(appId, timeStamp, nonceStr, packageValue, paySign ) {
        //调用微信支付（弹窗支付）
       var that = this;
        WeixinJSBridge.invoke(
           'getBrandWCPayRequest', {
               "appId":appId,     //公众号名称，由商户传入
               "timeStamp":timeStamp,         //时间戳，自1970年以来的秒数
               "nonceStr":nonceStr, //随机串
               "package":packageValue,
               "signType":"MD5",         //微信签名方式：
               "paySign":paySign //微信签名
           },function(res) {
               if(res.err_msg == "get_brand_wcpay_request:ok" ) {
                  that.submit();
               }else if(res.err_msg == "get_brand_wcpay_request:cancel" ){
                that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_RECHARGE_PAY_CANCEL_PREF});
                that.cancelOrder();                
                
                that.$dialog.notify({mes: '您已取消支付', icon: 'error', timeout: 1000});
                  
                  //that.optsRecharge.activityId = this.activityId;
                  //this.$store.dispatch('simulationPay', that.cancelOrder);
                  
                  //that.$router.go(0);
               }else {
                setSessionStore('s_1','payerror');
                  that.$router.push({
                    path:'/payerror',
                    query: {
                    err: res.err_msg
                  }});
                  that.$dialog.alert({mes: JSON.stringify(res)});
               }
               if(window.wxdebug == true) {
                  that.$dialog.alert({mes: JSON.stringify(res)});
               }
               that.isApply = false;
               // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。
           }
         );
      },
      tipShow() {         
        this.tips2=true;
        this.cardNumber=this.cardSelectNumber;
      },
      //取消弹框
      cancelCar(){
        this.tips2=false;
      },
      //确认选择会员卡
      carSubmit(){
        this.tips2=false;
        this.cardSelectNumber=this.cardNumber;
        this.cardSelectMoney=this.cardMoney;

        //选择完后需要提交到sessionStorege
        this.selectedVipCardInfo({"vipCard":this.cardSelectNumber, "balance":this.cardSelectMoney});
      },
      //会员卡选择
      carCheck(cardNumber,balance){
        this.cardNumber=cardNumber;
        this.cardMoney=balance;
      },
      //用户选择完 卡号后需要保存当前选择项，否则会出现选择完推荐人或者刷新当前页面后，之前选择的卡号被重置
      selectedVipCardInfo(param = {"vipCard":"", "balance":"0"}) {
        let selectedCardInfoStr = JSON.stringify({"vipCard":param.vipCard, "balance":param.balance});
        sessionStorage.setItem("recharge_selected_vipcard", selectedCardInfoStr);

        this.cardSelectNumber = param.vipCard;
        this.cardSelectMoney = param.balance;
      }
    }
  }
</script>
