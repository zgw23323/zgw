<template>
  <transition name="page">
    <section class="ck-doc information">
      <!-- <div :class="{'isShowType':!openType}"> -->
      <div class="information_apply">
        <div class="information-bd">
          <div class='item' :data-status="uNameStatus">          	 	           
  	        <span slot="left">姓名：</span>
  	    	<input slot="right" type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.name" placeholder="请输入您的姓名" maxlength="12">	         
          </div>
          <div class='item'>          	 	           
  	        <span slot="left">职位：</span>
  	    	<input slot="right" type="text" v-model="optApply.jobTitle"  @focus="handleFocus" v-on:blur="changeInput" placeholder="请输入您的职位" maxlength="16">	         
          </div>
          <div class='item'>          	 	           
  	        <span slot="left">手机号：</span>
  	    	<input slot="right" name='phone' v-model="phone"  @focus="handleFocus" v-on:blur="changeInput" type="text" placeholder="请输入您的手机号" maxlength="11">	         
          </div>
          <div class='item send-i'>
          	<input class='btn-left'  @focus="handleFocus" v-on:blur="changeInput" type="text" v-model='vCode' placeholder="请输入验证码">         	 	           
  	        <!-- <a :class='{"code-r": sendTrue}' href="javascript: void(0);" @click="sendCode">{{codeText}}</a> -->
            <!-- <yd-sendcode slot="right" v-model="start1" @click.native="sendCode" second="60"></yd-sendcode> -->
            <yd-sendcode slot="right" v-model="start1" @click.native="sendCode" second="60"></yd-sendcode>	         
          </div>        
          <section class="list">
      		<div @click='selectStore'>所属门店<a href="javascript: void(0);"><img src="../img/gt_03.png"></a></div>
      		<!-- <div>所属门店<img src="../img/gt_03.png"></div> -->
      		<div id="here">
      			<div v-for="(store, index) in pastStore"><span>{{store.orgShortName}}</span><span class="del" @click='deletStore(store.id)'><img src="../img/close_03.png"></span></div>
      		</div>
          <div class="more_store_btn" @click="moreStore" v-if="storeLen">查看更多<span></span></div>
  	     </section>
          
          </div>
        </div>
        <!-- <div :class="{'information-ft dis':isKeyUp,'information-ft':!isKeyUp}" v-if="openType" @click="submit"> 提交 </div> -->
        <div :class="{'information-ft dis information_apply':isKeyUp,'information-ft information_apply':!isKeyUp}"  @click="submit"> 提交 </div>
      </div>
      <!-- <div :class="{'isShowType':openType}"> -->
      <div id="information_result">
          <div class="apply-result-head">
            <div class="apply-result-icon"><img src="../img/nav_icon_apply_result@3x.png" /></div>
            <div class="apply-result-des">
              <span v-if="resultData.status ==0" class='apply-s'>审核中...</span>
              <span v-else-if="resultData.status ==1"class='apply-succes'>审核成功</span>
              <span v-else="resultData.status ==2" class='apply-fali'>审核失败</span>
            </div>
          </div>
          <div class='apply-result-info'>
              <div class='apply-title'>审核信息</div>
              <ul>
                 <li class="item1">                   
                    <div><span>姓名：</span>{{resultData.name}}</div>
                 </li>
                 <li class="item1">                   
                    <div><span>职业：</span>{{resultData.jobTitle}}</div>
                 </li>
                 <li class="item1">                   
                    <div><span>手机号：</span>{{resultData.phone}}</div>
                 </li>
                 <li class="item2">                   
                    <div><span>所属门店：</span></div>
                    <!-- <div class="apply-store-li"> -->
                      <!-- <span v-for="(store, index) in resultData.pushapplyOrgRels">{{store.orgShortName}}<i v-if="index!=pushapplyOrgRelsLen">、</i></span> -->
                      <!-- <span v-for="(store, index) in resultData.pushapplyOrgRels">{{store.orgShortName}}</span> -->
                    <!-- </div> -->
                 </li>
                 <li class="apply-store-li">
                      <span v-for="(store, index) in resultData.pushapplyOrgRels">{{store.orgShortName}}</span> 
                 </li>
              </ul> 
          </div>
          <div v-if="resultData.status ==2" class="register-ft">
            <div class="ck-btn">
              <div class="information-ft" @click="againApply">再次申请</div>
            </div>
          </div>
          <div v-if="resultData.status ==1" class="cancleApply-ft"><div class='information-ft' @click='cancelApply'>解绑</div></div>

          <div class='cancel_apply_tip' v-if="tips1">&nbsp;</div> 
          <div class='cancelApply-tips' v-if="tips1">
              <div class='close-tip' @click='tipsCancel'></div>
              <div class="tips-con">
                  <div class="tips-con-h">确认解绑吗?</div>
                  <div>解绑后不再收到消息推送哦</div>
              </div>
              <div class="tips-btn">
                <div class='tips-cancel' @click='tipsCancel'>取消</div>
                <div class="tips" @click='submitCancelApply'>确定</div>
              </div>
          </div>      
      </div>
    </section>
  </transition>
</template>
<style type="text/css">
  .dis{
    position: static;
  }
  .isShowType{
    display: none;
  }
  .information .send-i .btn {
    width: 33%;
  }
  #information_result,.information_apply {
    display: none;
  }
</style>
<script type="text/babel">
import { mapState } from 'vuex';
import {removeSessionStore,setSessionStore,getSessionStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
export default {

  data() {
    return {
     // title: '',
     openType:true,
	   phone: '',
	   showRefer: true,
	   phoneStatus:'',
	   vCodeStatus:'',
	   vCode:'',
     start1: false,
	   jobTitleStatus:'',
     uNameStatus: '',
	   cendTime:60,
	   sendTrue:true,
     isKeyUp:false,
     isAgainApply:'false',
	   codeText:'获取验证码',
	   optApply: {//发起支付前 要验证的参数
	      verifycode: '', //
	      phone:'', 
	      openId:'',
	      jobTitle:'',//职位
	      orgIds:'',//所选门店id集合       
	      name:''//姓名
	   },
	   optCode:{
	   		reqType:'apply',
	   		// type:'',
	   		phone:''
	   },
     isOpenStore:false,
     pastStore:[],//缓存部分门店
	   storeSelect:[],//存放所有门店
	   apiSendUrl: window.baseUrl+window.appName+'/loginapi/getVerifyCode?orgCode='+window.orgCode,//获取短信验证码
	   apiApplyUrl: window.baseUrl+window.appName+'/wxpay/addPushApply?orgCode='+window.orgCode,//申请授权
     apiApplyResultUrl: window.baseUrl+window.appName+'/wxpay/findPushApply?orgCode='+window.orgCode,
      apiCancelApplyUrl:window.baseUrl+window.appName+'/wxpay/unbindPushApply?sType=unbind&orgCode='+window.orgCode,
      optApplyResult:{//查询申请信息id
        openId:''
      },
      optCancelApply:{//取消绑定主键id
        id:''
      },
      tips1:false,
      storeLen:false,
      pushapplyOrgRelsLen:0,
      resultData:{}      
    }
  },
  watch: {
    // '$route.path': 'markInit',  //监听当前路由变化（辅助初始化）
    'optApply.name'() {
    	console.log("uNameValid", this.optApply);
        if(!this.uNameValid(this.optApply.name)) {  //监听当前输入用户名并实时校验
          this.uNameStatus = "success";
          this.setApplyInfo();
        }else {
          this.uNameStatus = "error";
        }
    },
    'optApply.jobTitle'() {
        if(!this.uPositionValid(this.optApply.jobTitle)) {  //监听当前输入职位并实时校验
          this.jobTitleStatus = "success";
          this.setApplyInfo();
        }else {
          this.jobTitleStatus = "error";
        }
    },
    phone() { //监听当前输入手机号并实时校验
        if(this.telInvalid(this.phone)) {
          this.phoneStatus = "success";
          this.optApply.phone = this.phone;//及时设置，便于缓存
          this.setApplyInfo();
        }else {
          this.phoneStatus = "error";
        }
    },
    vCode() { //监听当前输入验证码并实时校验
        if(this.vCodeInvalid(this.vCode)) {
          this.vCodeStatus = "success";
          this.optApply.verifycode = this.vCode;
          this.setApplyInfo();
        }else {
          this.vCodeStatus = "error";
        }
    }
  },
  mounted() {
    //挂载完毕初始化页面数据
      // document.title = this.title;
      
      //拿到当前路由
      let applyUrl = window.location.search.split('?')[1].split('&');
      let applyObj = {};
     for(var j=0;j<applyUrl.length;j++){
        var item = applyUrl[j].split('=');
        applyObj[item[0]] = item[1];
     }
      // let _newUrlStatus = applyObj.pushS;//拿到状态
      let apply_openId = applyObj.openId;
      let apply_again = sessionStorage.getItem('apply_again_open');
      if(apply_again!=''&& apply_again != null && apply_again != undefined){
        this.isAgainApply = apply_again;
      }
      // document.getElementById("body").setAttribute('style','background:#fff');
      //状态为3是申请授权,在此处进行判断是要知道进来的是什么模块
       
      this.optApply.openId = apply_openId;
      if(apply_openId !=''&&apply_openId !=null && apply_openId != undefined){
         this.optApplyResult.openId = apply_openId;
         this.fetchApplyInfo();
      }else{
        document.title = '完善申请信息';
        if(this.isAgainApply =='true'){
          this.setInformation();
        }else{
          // sessionStorage.removeItem("information_apply");//从缓存中清空填写的信息
          // sessionStorage.removeItem('information_selected_store');//没有拿到openId到申请页，清空缓存数据
          removeSessionStore("information_apply");
          removeSessionStore('information_selected_store');
        }      
        this.openType=true;
        //优化跳闪问题
        this.setInformationApplyDom('block');
        
      }
      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_PERFECTIONINFORMATION});
       
  },
  //添加钩子，监听路由变化时候跳转问题
  beforeRouteLeave(to, from, next){
     var apply_statu = sessionStorage.getItem('apply_result_statu');
     // console.log('to->',to,'form->',form,'next->');
    if(apply_statu !=null && apply_statu !=undefined && apply_statu !=''){
      //此处判断的逻辑是，申请结果状态，并且不是再次申请，从门店选择回来到申请页时退出到微信菜单，否则正常跳转路由
      if(apply_statu=='0'||apply_statu=='1' || (apply_statu=='2' && this.isAgainApply=='false')){
        WeixinJSBridge.call('closeWindow');       
        next(false);
      }else{
        next();
      }
    }    
  },
  methods: {
    submit() {
       let that = this;
        //提交数据
        if(this.vCodeStatus =="success" && this.phoneStatus == "success" && this.uNameStatus == 'success' && this.jobTitleStatus == 'success' && this.storeSelect.length >0) {

          this.optApply.verifycode = this.vCode;
          this.optApply.phone = this.phone;
          this.optApply.orgIds = this.getId();
          // this.apiApplyUrl.type = 'valid';
          this.$dialog.loading.open('保存中...');
          this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_PERFECTIONINFORMATION_BTN_APPLY_SAVE});
          this.$http.post(this.apiApplyUrl,this.optApply)
            .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              //申请成功后缓存用户openId，供申请结果详情
              setSessionStore('apply_again_open',that.isAgainApply);
              setSessionStore('apply_result_statu',0);//缓存状态码，点击后退时需要,提交成功去到审核状态
              that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
                document.title = '审核状态';
                that.openType = false; 
                that.setInformationApplyDom('none');               
                that.optApplyResult.openId = data.var; 
                that.fetchApplyInfo();            
              }});
              that.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_PERFECTIONINFORMATION_MESS_APPLY_OK});
            }else {
              if(data.msg == "verifycode is error"){
                that.$dialog.notify({mes: '验证码错误', icon: 'error', timeout: 1200});
              }
              if(data.code == "PUSHAPPLY_Q_FAL_0X003"){
                that.$dialog.notify({mes: '申请失败，请重新申请', icon: 'error', timeout: 1200});
              }
              that.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_PERFECTIONINFORMATION_MESS_APPLY_FAIL});
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
            },300);
          });

        }else {
           //校验输入的姓名
  	      if(this.uNameStatus !="success") {
  	        this.$dialog.notify({mes: '请输入您的姓名', icon: 'error', timeout: 1200});
  	        return
  	      }
	       //职位校验
          if(this.jobTitleStatus !='success'){
          	this.$dialog.notify({mes: '请输入您的职位',icon: 'error', timeout: 1200});
          	return
          }
          if(this.phoneStatus != "success") {
            this.$dialog.notify({mes: '输入手机号错误', icon: 'error', timeout: 1200});
            return;
          }
          if(this.vCodeStatus != "success") {
            this.$dialog.notify({mes: '请输入6位验证码', icon: 'error', timeout: 1200});
            return;
          }
          if(this.storeSelect.length ==0){
            this.$dialog.notify({mes: '请选择所属门店', icon: 'error', timeout: 1200});
            return;
          }
         
        }
    },
    //优化闪跳方法
    setInformationApplyDom(pType){
      var information_dom = document.getElementsByClassName("information_apply");
       information_dom[0].style.cssText='display:'+pType;
       information_dom[1].style.cssText='display:'+pType;
        if(pType == 'block'){
          document.getElementById("information_result").setAttribute('style','display:none'); 
        }else{
          document.getElementById("information_result").setAttribute('style','display:block'); 
        }
    },
    setInformation(){
    	//从sessionStorage中加载已经填写的信息
        let InfoStr = sessionStorage.getItem("information_apply");
        let storeStr = sessionStorage.getItem('information_selected_store');
        
        //从缓存中取回填写的信息
        if (InfoStr != null && InfoStr != undefined && InfoStr != '') {
          let applyInfo = JSON.parse(InfoStr);          
          this.phone = applyInfo.phone;
          this.optApply.name = applyInfo.name;
          this.optApply.jobTitle = applyInfo.jobTitle;
          this.vCode = applyInfo.verifycode;
        }
        //从缓存中拿到选择的门店
        if(storeStr !=null && storeStr !=undefined){
          storeStr = JSON.parse(sessionStorage.getItem('information_selected_store'));

        	this.storeSelect = storeStr;
          for(var i=0;i<storeStr.length;i++){
            if(i<6){
              this.pastStore.push(this.storeSelect[i]);
            }            
          }
          if(storeStr.length>6){
            this.storeLen = true;
          }else{
            this.storeLen = false;
          }
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
          this.cendTime=60;
          this.optCode.phone = this.phone;
          this.$store.dispatch('log', {account:LOGCODE.HOME.BTN_SENDCODE_GET});
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
                this.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
              },300);
            });
        }

    },
    uNameValid(value,allowEmptyString) { //校验用户名方法
        let val= value.replace(/\s/g,"");
        return (val === null) || (val === undefined)
          || (!allowEmptyString ? val === '' : false)
          || (val.length === 0);
    },
    telInvalid(val) { //电话号码校验
        return /^[0-9]{11}\d*$/.test(val);
    },
    vCodeInvalid(val) { //验证码6位校验
        return /^[0-9]{6}\d*$/.test(val);
    },
    uPositionValid(value,allowEmptyString) {//校验职位方法
    	let val= value.replace(/\s/g,"");
        return (val === null) || (val === undefined)
          || (!allowEmptyString ? val === '' : false)
          || (val.length === 0);	
    },
    selectStore() {
        // this.storeRechargeInfo();
        //跳去地址
        this.isAgainApply = 'true';
        // sessionStorage.setItem('apply_again_open','true');
        setSessionStore('apply_again_open','true');
        this.$router.push({
          path: '/store',
          query: {
            from: 'information'
          }          
        });
    },
     
    //修改信息后设置缓存，确保选择完门店后可以回显
    setApplyInfo(){
    	// let InfoStr = JSON.stringify(this.optApply);
        // sessionStorage.setItem("information_apply", InfoStr);
      setSessionStore("information_apply",this.optApply);  
    },
    //删除门店
    deletStore(pId){
        let noChoosedArr = this.storeSelect.filter(function(data){// 
      		return data.id != pId;
    	});
      this.storeSelect = noChoosedArr;     
      if(this.isOpenStore){
         this.pastStore = this.storeSelect
      }else{
        this.pastStore = [];
        for(var i=0;i<this.storeSelect.length;i++){
            if(i<6){
              this.pastStore.push(this.storeSelect[i]);
            }            
        }
      }
      
      if(this.storeSelect.length>6 && this.pastStore.length != this.storeSelect.length){
        this.storeLen = true;
      }else{
        this.storeLen = false;
      }
    	// sessionStorage.setItem("information_selected_store", JSON.stringify(this.storeSelect));
      setSessionStore("information_selected_store",this.storeSelect);
    },
    handleFocus(){
      this.isKeyUp = true; 
      var that = this;
      setTimeout(function(){
          that.isKeyUp = true;
        },100);    
    },
    changeInput(){
      var that = this;
      setTimeout(function(){
          that.isKeyUp = false;
      },100);      
    },
    //点击更多查看门店
    moreStore(){
      this.pastStore = this.storeSelect;
      this.storeLen = false;
      this.isOpenStore = true;
    },
    //获取id字符串
    getId(){
    	let _str = [];
    	this.storeSelect.filter(function(data){// 
    		_str.push(data.id);
      		// return data.id;
    	});
    	return _str.join(',');    	 
    },
    fetchApplyInfo(){
          //获取申请页面数据
      let that = this;

      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.apiApplyResultUrl,that.optApplyResult)
        .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          // that.tips1 = false;
          if(data.code == "S_OK") {
            let apply_status = data.var.status;
            // sessionStorage.setItem('apply_result_statu',apply_status);
            setSessionStore('apply_result_statu',apply_status);
            if(apply_status != 0 && apply_status !=1 && apply_status !=2 || (apply_status ==2 && that.isAgainApply =='true')){
               document.title = '完善申请信息';
               if(that.isAgainApply =='true'){
                  that.setInformation();
               }else{
                  // sessionStorage.removeItem('information_selected_store');
                  // sessionStorage.removeItem("information_apply");//从缓存中清空填写的信息
                  removeSessionStore('information_selected_store');
                  removeSessionStore("information_apply");//从缓存中清空填写的信息
               }            
               that.openType=true;
               that.setInformationApplyDom('block');
               // that.optApply.openId = data.var.openId;
            }else{
                document.title = '审核状态';
                that.openType=false;
                that.setInformationApplyDom('none');
                that.resultData = data.var;
                that.pushapplyOrgRelsLen = data.var.pushapplyOrgRels.length-1;
                that.optCancelApply.id = data.var.id;            
            }      
          }else if(data.code =="PUSHAPPLY_Q_FAL_0X002"){
               document.title = '完善申请信息';
               if(that.isAgainApply =='true'){
                  that.setInformation();
               }else{
                  // sessionStorage.removeItem('information_selected_store');
                  // sessionStorage.removeItem("information_apply");//从缓存中清空填写的信息
                  removeSessionStore('information_selected_store');
                  removeSessionStore("information_apply");//从缓存中清空填写的信息
               }            
               that.openType=true;
               that.setInformationApplyDom('block');
          }else {
            // that.tips1 = true;
            this.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
          }
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    againApply(){
      //跳去再次申请页面
      document.title = '完善申请信息';
      this.openType=true;
      this.setInformationApplyDom('block');
      this.storeLen = false;
      this.isAgainApply = 'true';
       
      removeSessionStore("information_apply");
      setSessionStore('apply_again_open',this.isAgainApply);
      removeSessionStore('information_selected_store');//从缓存中清空填写的信息
    },
    cancelApply(){
          //获取个人中心页面数据
       this.tips1 = true;
    },
    submitCancelApply(){
        //申请解绑
      let that = this;
      that.tips1 = false;
      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.apiCancelApplyUrl,that.optCancelApply)
        .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          // that.tips1 = false;
          if(data.code == "S_OK") {
             this.$dialog.notify({mes: "解绑成功", icon: 'error', timeout: 2000}); 
             setTimeout(()=>{
                WeixinJSBridge.call('closeWindow');
             },2000)       
            this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_PERFECTIONINFORMATION_MESS_CANCEL_APPLY_OK});
          }else {
            that.tips1 = true;
            this.$dialog.notify({mes: "网络异常，请稍后再来解绑~", icon: 'error', timeout: 1000});
          }
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    tipsCancel(){
      this.tips1 = false;
    }
  }
}
</script>
