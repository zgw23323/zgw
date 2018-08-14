<template>
  <transition name="page">
    <section class="information infoPushApply applyResult" v-if="showType =='3'">
      <div id="user_logo" class="user_logo">
        <input type="file" id='workerLogo'  @change="readFile('workerLogo')" accept="image/*" capture="camera">
        <img :src="logopath" v-if="logopath !=''" />
        <img v-else src="../../img/serviceImg/user_logo.png"/>
      </div>
      <div id="information-bd" class="information-bd">
        <div class='item' :data-status="uNameStatus">                        
          <span slot="left">姓名：</span>
          <input slot="right" type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.name" placeholder="请输入您的姓名" maxlength="12">           
        </div>
        
        <div class='item'>                         
          <span slot="left">手机号：</span>
          <input slot="right" name='phone' @focus="handleFocus" v-on:blur="changeInput" v-model="phone" type="text" placeholder="请输入您的手机号" maxlength="11">          
        </div>
        <div class='item send-c'>
          <div style="width: 60%;">
            <span slot="left">验证码：</span>
            <input placeholder="请输入验证码" @focus="handleFocus" v-on:blur="changeInput" style="margin-left:.5rem;width:50%;" class="has_width" type="text" v-model='vCode'/>
          </div>                       
          <yd-sendcode slot="right" style="padding:0 .1rem;" v-model="start1" @click.native="sendCode" second="60"></yd-sendcode>           
        </div>         
        <div class="item s_con" @click="tipOpen(2)">
          <div>
            <span>门店：</span>
            <input type="text" class="has_width"  placeholder="请选择门店" readonly="readonly" v-model="storeName" />
          </div>
          <div><i class="right">&nbsp;</i></div>
        </div>
        <div class='item'>                         
          <span slot="left">职位：</span>
          <input slot="right" type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.jobTitle" placeholder="请输入您的职位" maxlength="16">          
        </div>
        <div class="p_top_1">
          <div>申请开通服务：</br>
            <ul class="pepair_parts">
              <li :class="{'curBtn':serviceOpt.service1 !=''}" @click="openServiceBtn('BLBZKD','1')"><span>玻璃单独保障服务</span></li>
              <li :class="{'curBtn':serviceOpt.service2 !=''}" @click="openServiceBtn('BLBZSP','2')"><span>玻璃保修服务</span></li>
              <li :class="{'curBtn':serviceOpt.service3 !=''}" @click="openServiceBtn('CZTSTZ','3')"><span>充值推送通知</span></li>
              <li :class="{'curBtn':serviceOpt.service4 !=''}" @click="openServiceBtn('MDCW','4')"><span>门店财务</span></li>
              <li :class="{'curBtn':serviceOpt.service5 !=''}" @click="openServiceBtn('SPRINGWARM','5')"><span>春节送温暖</span></li>
              <li :class="{'curBtn':serviceOpt.service6 !=''}" @click="openServiceBtn('SPRINGWARM','6')"><span>预约维修服务</span></li>
            </ul>
          </div>
        </div>
      </div>
      <div id="information-ft" :class="{'information-ft dis':isKeyUp,'information-ft':!isKeyUp}" @click="submit">申请开通</div>
      <!-- <div id="information-ft" :class="{'information-ft dis information_apply':isKeyUp,'information-ft information_apply':!isKeyUp}"class='infoPushApply_subBtn' @click="submit">申请开通</div> -->
        <!-- 选择门店 -->
      <div class='tip_f' v-if="tips2" v-model="tips2" @click="tipOpen(2)">&nbsp;</div>
      <div class='ck-tipsFormat center-tip service_cancel_tip' v-model="tips2" v-if="tips2">
        <section class="service_cancel">
          <p>申请绑定的门店</p>
          <div class="service_li max_height">
              <div :class="{'has': item.isSelect}" v-for="(item,index) in storeList" @click= "storeBtn(item,index)">{{item.orgShortName}}</div>
          </div>
          <div class="bottom_btn fiex_tip">
            <div @click="tipOpen(2)">取消</div>
            <div @click="tipOpen(2)">确定</div>
          </div>
        </section>  
      </div>
    </section>
    <section class="workerHome" v-if="showType =='1'">
        <article id="work_header" class="header">
          <div class="item_left">
              <div class="logo">
              <img :src="resultData.logopath" v-if="resultData.logopath !='' && resultData.logopath !=null" />
              <img v-else src="../../img/nav_icon_me_nor@3x.png"/></div>
              <div class="info">
                <p class="p1">您好！<span class="i_name">{{resultData.name}}</span></p>
                <p class="p2">职位：{{resultData.jobTitle}}</p>
              </div>
          </div>
          <div class="item_right" @click="goToRouter('workerInfo')">查看资料</div>
        </article>

        <article id="work_content" class="content">
            <div class="item" v-for="service,index in serviceList">
              <p class="s_menu_name">{{service.serviceName}}</p>
              <p  class='s_menu_mess p_bottom_1' v-html="service.messsage"></p>
              <div  class="s_menu_btn btn_2" v-if="service.serviceStatu=='1'" @click="goToRouter('applyResult',service)">申请中...</div>
              <div  class="s_menu_btn btn_1" v-else-if="service.serviceStatu=='2' && service.serviceCode=='BLBZKD'" @click="goToRouter('glassOpenOrder',service)">车主购买</div>
              <div  class="s_menu_btn btn_1" v-else-if="service.serviceStatu=='2' && service.serviceCode=='BLBZSP'" @click="goToRouter('seachRecord',service)">我要保修</div>
              <div  class="s_menu_btn btn_1" v-else-if="service.serviceStatu=='2' && service.serviceCode=='CZTSTZ'" @click="goToRouter('applyResult',service)">已开通</div>
              <div  class="s_menu_btn btn_1" v-else-if="service.serviceStatu=='2' && service.serviceCode=='MDCW'" @click="goToRouter('storeAccounting',service)">已开通</div>
              <div  class="s_menu_btn btn_1" v-else-if="service.serviceStatu=='2' && service.serviceCode=='APPOINT'" @click="goToRouter('appointmentService',service)">预约维修</div>
              <div  class="s_menu_btn btn_1" @click="tips3=true;" v-else-if="service.serviceStatu=='2' && service.serviceCode=='SPRINGWARM'">验证</div>
              <div  class="s_menu_btn btn_3" v-else @click="goToRouter('applyResult',service)">申请开通</div>
            </div>
        </article>
        <div id="work_btn" class="workerHome_btn w_btn_1" @click="tipOpen(1)" v-if="isShowUnbind"><span>我要解绑</span></div>
        
        <!--解绑弹窗-->
      <yd-popup v-model="tips1" position="center" class="service_cancel_tip" height="50%" width="92%">
        <article class="service_cancel">
            <p>解除绑定</p>
            <div class="service_li">
              <div v-for="ser,index in serviceList" :class="{'has':ser.serviceStatu=='2' && !ser.isSelect,'isCheck':ser.serviceStatu=='2' && ser.isSelect}" @click="cancelBtn(ser,index)"><span>{{ser.serviceName}}</span>
                <span :class="{'ser_statu has_cancel':!ser.isSelect,'ser_statu': ser.isSelect}" v-if="ser.serviceStatu=='2'">已绑定</span>
                <span class="ser_statu" v-else-if="ser.serviceStatu=='1'">申请中</span>
                <span class="ser_statu" v-else>未绑定</span>
              </div>
            </div>
            <div class="bottom_btn">
              <div @click="tipOpen(1)">取消</div>
              <div @click="submitCancelApply">确定</div>
            </div>
        </article> 
      </yd-popup>

      <!--选择验证类型-->
      <yd-popup v-model="tips3" position="bottom" class="ser-legalfield" height="2.66rem">
        <ul class="code_btn">
          <li @click="setTmestamp"><span>扫一扫</span></li>
          <li @click="codeBtn('code')"><span>验证码</span></li>
          <li @click="tips3=false;"><span>取消</span></li>
        </ul>
      </yd-popup>

       <!--扫码弹窗内容-->
      <yd-popup v-model="tips5" position="center" width="5.8rem">
        <div class="p_tip">
          <div class="close_i" @click="closeTip(5)"><i>&nbsp;</i></div>
           <div class="i_info">
            <div class="i_img"><img :src="imgBaseUrl+productData.thumbnailUrl" /></div>
            <p>{{productData.prodTitle}}<span class="has" v-if="productData.isFirst==false">（已兑换）</span><span class="has" v-if="productData.isFirst ==true && productData.redeemStatus==2">（兑换成功）</span></p>
          </div>
        </div>
      </yd-popup>
    </section>
      
  </transition>
</template>
<script type="text/babel">
import { mapState } from 'vuex';
import {removeSessionStore,setSessionStore,getSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
export default {
  data() {
    return {
      isWx:true,
      tips1:false,
      tips2:false,
      tips4: false,
      tips3: false,
      tips5: false,
      showType:'',
      phone: '',
      phoneStatus:'',
      vCodeStatus:'',
      vCode:'',
      start1: false,
      jobTitleStatus:'',
      uNameStatus: '',
      sendTrue:true,
      isKeyUp:false,
      storeList:[],
      logopath:'',
      isShowUnbind:true,
      serviceList:[
        {
          messsage:'帮助客户开通玻璃单独</br>保障服务',
          serviceCode:'BLBZKD',
          serviceName:'玻璃单独保障服务',
          serviceStatu:'0',
          isSelect:false
        },
        {
          messsage:'工作人员提交车主</br>保修信息',
          serviceCode:'BLBZSP',
          serviceName:'玻璃保修服务',
          serviceStatu:'0',
          isSelect:false
        },
        {
          messsage:'通过微信公众号接收</br>客户充值信息',
          serviceCode:'CZTSTZ',
          serviceName:'充值短信通知',
          serviceStatu:'0',
          isSelect:false
        },
        {
          messsage:'随时查看门店财务</br>收支情况',
          serviceCode:'MDCW',
          serviceName:'门店财务',
          serviceStatu:'0',
          isSelect:false
        },
        {
          messsage:'扫码或输入验证码确认</br>车主抢购的商品',
          serviceCode:'SPRINGWARM',
          serviceName:'春节送温暖活动',
          serviceStatu:'0',
          isSelect:false
        },
        {
          messsage:'通过微信公众号接收</br>客户预约申请信息',
          serviceCode:'APPOINT',
          serviceName:'预约维修服务',
          serviceStatu:'0',
          isSelect:false
        }
      ],
      optApply: {//发起支付前 要验证的参数
        verifycode: '', //
        phone:'', 
        logopath:'',
        isUpload:false,
        openId:window.openId,
        jobTitle:'',//职位
        orgId:'',//所选门店id集合       
        name:'',//姓名
        applyServers:[]//开通的服务
      },
      serviceOpt:{//0未申请;1申请中;2已通过;3已拒绝;4已解绑;5已取消;默认为0,
        service1:'',
        service2:'',
        service3:'',
        service4:'',
        service5:'',
        service6:''
      },
      optCode:{
        reqType:'apply',
        phone:''
      },
      cancelServiceOpt: {
        type:'',
        value:'',
        ind:0
      },
      storeName:'',
      storeSelect:[],//存放所有门店
      imgBaseUrl: window.imgBaseUrl,
      apiSendUrl: window.baseUrl+window.appName+'/loginapi/getVerifyCode?orgCode='+window.orgCode,//获取短信验证码
      apiApplyUrl: window.baseUrl+window.appName+'/wxpay/addPushApply?orgCode='+window.orgCode,//申请授权
      apiApplyResultUrl: window.baseUrl+window.appName+'/wxpay/findPushApply?orgCode='+window.orgCode,
      apiUnbindUrl:window.baseUrl+window.appName+'/wxpay/unbind/pushApply',//解除绑定
      storeApiUrl: window.baseUrl+window.appName+'/wxpay/listOrgs?orgCode='+window.orgCode,//门店列表
      optApplyResult:{//查询申请信息id
        openId: window.openId
      },
      apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=glass&isBase64=y&flag=thumb',
      optCancelApply:{},//取消绑定主键id
      storeLen:false,
      pushapplyOrgRelsLen:0,
      wxCodeOpt:{},
      apiUrl: window.baseUrl+window.appName+'/shopapi/func/springwarm/wxYiyuanRedeem?sid='+window.shopsid+'&openId='+window.openId,
      apiCodeUrl:window.baseUrl+window.appName+"/wxpay/jsapi?targetURL=",//微信扫一扫获取签名
      productData:{},
      resultData:{
        "id":"",
        "phone":"",
        "name":"",
        "jobTitle":"",
        "nickname":"",
        "logopath":"",
        "corpId":"",
        "orgId": "",
        "orgShortName":"",
        "hadFlag":false,
        "servers":[],
        "orgCode":""
      }
    } 
  },
  watch: {
    '$route.path': 'markInit',  //监听当前路由变化（辅助初始化）
    'optApply.name'() {
        if(!ydFunLib.uNameValid(this.optApply.name)) {  //监听当前输入用户名并实时校验
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
        if(ydFunLib.telInvalid(this.phone)) {
          this.phoneStatus = "success";
          this.optApply.phone = this.phone;//及时设置，便于缓存
          this.setApplyInfo();
        }else {
          this.phoneStatus = "error";
        }
    },
    vCode() { //监听当前输入验证码并实时校验
        if(ydFunLib.vCodeInvalid(this.vCode)) {
          this.vCodeStatus = "success";
          this.optApply.verifycode = this.vCode;
          this.setApplyInfo();
        }else {
          this.vCodeStatus = "error";
        }
    }
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
      this.fetchApplyInfo();
      this.fetchStoreInfo();
      // this.setScrollHeight();
      this.isWx = ydFunLib.curNavigator();
      removeSessionStore('s_1');
      this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_INFOPUSHAPPLY});
       
  },
  methods: {
    tipOpen(tip){
      this['tips'+tip] = !this['tips'+tip];
    },
     //设置滑动区域的最大高度
    setScrollHeight(pDomLen,pNum){
      var num = 0;
      var bodyHeight = document.body.offsetHeight;
      for(var i=0;i<pDomLen.length;i++){
        num = num + document.getElementById(pDomLen[i]).offsetHeight;
      }
      var _height = bodyHeight-num-pNum;
       _height = _height>0?_height:_height.toString().substring(1);
       document.getElementById(pDomLen[pDomLen.length-2]).style='margin-bottom:'+_height+'px';
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


    goToRouter(str,opt){
      this.setApplyInfo();
      setSessionStore('s_1',str);
      if(str == 'workerInfo' || str == 'seachRecord'){
          this.$router.push({
            path: '/'+str
          });
      }else if(str == 'appointmentService'){
        window.location.href = window.baseUrl+'/shop/wxpay/add/pushapply?openId='+window.openId+'&module=appointmentService';
      }else {
        this.$router.push({
          path: '/'+str,
          query:{'serverCode':opt.serviceCode,'value':opt.serviceStatu,'applyId':this.resultData.id}
        });
      }
    },
    //获取本地图片信息
    readFile(str){ 
      var that = this;
      var file = document.getElementById(str).files[0]; //获取file对象
      //判断file的类型是不是图片类型。
      if(!/image\/\w+/.test(file.type)){ 
          alert("文件必须为图片！"); 
          return false; 
      }
      var reader = new FileReader(); //声明一个FileReader实例
      reader.readAsDataURL(file);
      reader.onload = function(e){ 
        ydFunLib.appendFile(this.result,function(base64){
            that.prodPicUpload(base64,str);
        });
      } 
    },
    //上传图片
    prodPicUpload(base64,str){
      var that = this;
      this.$dialog.loading.open('上传中...');
      this.tips4 = true;
      that.$http.post(this.apiUploadFileUrl, JSON.stringify({"imgStr":base64}))
          .then((response) => {
            const data = response.data;
            console.log('data=>',data);
            if(data.code == "S_OK") {
              var fileId = data["var"];
                if(fileId){
                  that.optApply.logopath = fileId;
                  that.logopath = that.imgBaseUrl+'/shop/downLoad/showLogo/'+fileId+'?type=thumb';
                  that.optApply.isUpload = true;
                }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
          },(response) => {
            that.$dialog.loading.close();
            that.tips4 = false;
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
    },
    //门店点击
    storeBtn(pObj,pIndex) {
      this.storeList[pIndex].isSelect = !this.storeList[pIndex].isSelect;
      for(var i=0;i<this.storeList.length;i++){
        if(this.storeList[i].id != pObj.id){
          this.storeList[i].isSelect = false;
        } 
      }
      if(this.storeList[pIndex].isSelect){
        this.optApply.orgId = pObj.id;
        this.storeName = pObj.orgShortName;
      }else{
        this.optApply.orgId = '';
        this.storeName = '';
      }
    },
    //获取门店列表请求方法
    fetchStoreInfo(){
      let that = this;
      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.storeApiUrl)
        .then((response) => {
          const data = response.data;
          if (data.code === 'S_OK') {
            //这里使用双循环判断是因为要处理缓存中已经选择过的列表项
            for(var i=0;i<data.var.length;i++){
              var _i = data.var[i];
               data.var[i].isSelect=false;
            }
            that.storeList = data.var;
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
       let that = this;
       for(var key in this.serviceOpt){
        if(this.serviceOpt[key] !=''){
          this.optApply.applyServers.push(this.serviceOpt[key]);
        }
       }
        //提交数据
        if(this.vCodeStatus =="success" && this.phoneStatus == "success" && this.uNameStatus == 'success' && this.jobTitleStatus == 'success' && this.optApply.orgId !='' && this.optApply.applyServers.length >0) {

          this.optApply.verifycode = this.vCode;
          this.optApply.phone = this.phone;
          this.$dialog.loading.open('保存中...');
          this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.BTN_PAGE_INFOPUSHAPPLY_APPLY_SAVE});
          this.$http.post(this.apiApplyUrl,this.optApply)
            .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              //申请成功后缓存用户openId，供申请结果详情
              that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
                document.title = '服务权限';
                removeSessionStore('infoPush_selected_service');
                that.fetchApplyInfo();            
              }});
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_INFOPUSHAPPLY_APPLY_SAVE_OK});
            }else {
              if(data.msg == "verifycode is expired"){
                that.$dialog.notify({mes: '验证码错误', icon: 'error', timeout: 1200});
              }
              if(data.code == "PUSHAPPLY_Q_FAL_0X003"){
                that.$dialog.notify({mes: '申请失败，请重新申请', icon: 'error', timeout: 1200});
              }
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_INFOPUSHAPPLY_APPLY_SAVE_FAIL});
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });

        }else {
           //校验输入的姓名
          if(this.uNameStatus !="success") {
            this.$dialog.notify({mes: '请输入您的姓名', icon: 'error', timeout: 1200});
            return
          }
          if(this.phone==''){
            this.$dialog.notify({mes: '请输入您的手机号', icon: 'error', timeout: 1200});
            return
          }else if(this.phoneStatus != "success") {
            this.$dialog.notify({mes: '输入手机号错误', icon: 'error', timeout: 1200});
            return;
          }
          if(this.vCodeStatus != "success") {
            this.$dialog.notify({mes: '请输入6位验证码', icon: 'error', timeout: 1200});
            return;
          }
          if(this.optApply.orgId ==''){
            this.$dialog.notify({mes: '请选择所属门店', icon: 'error', timeout: 1200});
            return;
          }
         //职位校验
          if(this.jobTitleStatus !='success'){
            this.$dialog.notify({mes: '请输入您的职位',icon: 'error', timeout: 1200});
            return
          }
          if(this.optApply.applyServers.length==0){
            this.$dialog.notify({mes: '请选择申请开通的服务', icon: 'error', timeout: 1200});
            return;
          }
        }
    },
    //解绑
    cancelBtn(opt,pIndex){
      if(opt.serviceStatu=='2') {
          this.serviceList[pIndex].isSelect = !this.serviceList[pIndex].isSelect;
      }else{
        return;
      }
    },
    
    //开通服务选择
    openServiceBtn(str,n){
      var _str = this.serviceOpt['service'+n];
      this.serviceOpt['service'+n] = _str==''?str:'';
      console.log('this.serviceOpt',this.serviceOpt);
    },
    setInformation(){
      //从sessionStorage中加载已经填写的信息
        let infoOpt = getSessionStore("information_apply",true);
        let storeOpt = getSessionStore('information_selected_store',true);
        let serviceType = getSessionStore('infoPush_selected_service' ,true);
        
        //从缓存中取回填写的信息
        if (typeof(infoOpt) !='boolean') {
          this.phone = infoOpt.phone;
          this.optApply.name = infoOpt.name;
          this.optApply.jobTitle = infoOpt.jobTitle;
          this.vCode = infoOpt.verifycode;
        }
        //从缓存中拿到选择的门店
        if(typeof(storeOpt) !='boolean'){
          this.storeSelect = storeOpt;
          if(this.storeSelect.length>1){
            this.storeName = storeOpt[0].orgShortName+'...';
          }else if(this.storeSelect.length==1){
            this.storeName = storeOpt[0].orgShortName;
          }
        }

        //从缓存中拿到选择的服务项
        if(typeof(serviceType) !='boolean'){
          this.serviceOpt = serviceType;
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
          this.optCode.phone = this.phone;
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
    uPositionValid(value,allowEmptyString) {//校验职位方法
      let val= value.replace(/\s/g,"");
        return (val === null) || (val === undefined)
          || (!allowEmptyString ? val === '' : false)
          || (val.length === 0);  
    },
    selectStore() {
        //跳去地址
        this.setApplyInfo();
        this.$router.push({
          path: '/store',
          query: {
            from: 'infoPushApply'
          }          
        });
    },
    codeBtn(str){
      setSessionStore('s_1',str);
      this.$router.push({
        path: '/scanCode',
        query: {type:str}
      });
    },
    //修改信息后设置缓存，确保选择完门店后可以回显
    setApplyInfo(){
      setSessionStore("information_apply",this.optApply);
      setSessionStore('infoPush_selected_service',this.serviceOpt);
      setSessionStore('information_selected_store',this.storeSelect); 
    },
    
    //获取id字符串
    getId(){
      let _str = [];
      this.storeSelect.filter(function(data){
        _str.push(data.id);
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
          if(data.code == "S_OK") {
            if(data.var.hadFlag==true){
              document.title = '服务权限';
              this.showType = "1";
              that.resultData = ydFunLib.ydExtend(that.resultData,data.var,true);
              console.log('that.resultData',that.resultData);
              for(var i=0;i<that.resultData.servers.length;i++){
                var item = that.resultData.servers[i];
                for(var j=0;j<that.serviceList.length;j++){
                  if(item.serverCode == that.serviceList[j].serviceCode){
                    that.serviceList[j].serviceStatu = item.serverStatus;
                  }
                }
              }
              var len = that.resultData.servers.filter(function(data){
                return data.serverStatus == 2;
              });
              that.isShowUnbind = len.length>0? true:false;
              setSessionStore('workerInfo', that.resultData);

            }else{
              document.title = '申请开通服务权限';
              this.showType = "3";
            }
            if(data.var.logopath){
              var _logo = data.var.logopath.split('http:');
              if(_logo.length>1){
                that.resultData.logopath = data.var.logopath;
                that.logopath = data.var.logopath;
              }else{
                that.resultData.logopath = that.imgBaseUrl+'/shop/downLoad/showLogo/'+data.var.logopath+'?type=thumb';
                that.optApply.logopath = data.var.logopath;
                that.logopath = that.imgBaseUrl+'/shop/downLoad/showLogo/'+data.var.logopath+'?type=thumb';
              }
            }
          }else {
            this.$dialog.notify({mes: data.code, icon: 'error', timeout: 1000});
          }
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        },(response) => {
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    
    submitCancelApply(){
        //申请解绑
      let that = this;
      var str = [];
      var len = this.serviceList.filter(function(data){
        if(data.isSelect){
          str.push(data.serviceCode);
        }
        return data.isSelect;
      });
      if(len.length==0){
        this.$dialog.notify({mes: '请选择解绑的服务', icon: 'error', timeout: 1200});
        return;
      }
      that.optCancelApply.applyId=that.resultData.id;
      that.optCancelApply.serverCode = str;
      that.$dialog.loading.open('加载数据中...');
      that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.BTN_PAGE_INFOPUSHAPPLY_APPLY_UNBIND});
      that.$http.post(that.apiUnbindUrl,that.optCancelApply)
        .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            for(var i=0;i<that.serviceList.length;i++){
              if(that.serviceList[i].isSelect){
                that.serviceList[i].serviceStatu = '0';
              }
            }
            this.$dialog.notify({mes: "解绑成功", icon: 'error', timeout: 1500,callback: () => {
              var _opt = that.serviceOpt; 
              that.tipOpen(1);
            }});
            
            this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_INFOPUSHAPPLY_APPLY_UNBIND_OK});
          }else {
            this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_INFOPUSHAPPLY_APPLY_UNBIND_FAIL});
            this.$dialog.notify({mes: "网络异常，请稍后再来解绑~", icon: 'error', timeout: 1000});
          }
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        },(response) => {
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    closeTip(n){
      this['tips'+n]= false;
      this.setTmestamp();
    },
    //验证扫一扫签名是否过期
    setTmestamp() {
      var curT = Math.floor(new Date().getTime() / 1000);
      if(this.wxCodeOpt.timestamp&&(parseInt(this.wxCodeOpt.timestamp)+3600>curT)){//未过期
        this.scanQRCodeViter();
      }else{//已过期
        this.fetchSaoCode();
      }
    },
    //获取微信扫一扫签名
    fetchSaoCode() {
      let that = this;
      var targetURL = location.href.split('#')[0];
      that.$http.post(that.apiCodeUrl+encodeURIComponent(targetURL))
        .then((response) => {
          const data = response.data;
          if (data.code === 'S_OK') {
            that.wxCodeOpt = data.var;
            that.scanQRCodeViter();
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
    //扫一扫验证
    scanQRCodeViter() {
      let that = this;
      wx.config({
        debug: false, 
        appId: 'wx122c017d1608c62a',
        timestamp: that.wxCodeOpt.timestamp, //生成签名的时间戳
        nonceStr: that.wxCodeOpt.noncestr, //生成签名的随机串
        signature: that.wxCodeOpt.signature,
        jsApiList: ['scanQRCode']
      });
      wx.scanQRCode({
        desc: 'scanQRCode desc',
        needResult: 1, // 默认为0，扫描结果由微信处理，1则直接返回扫描结果，
        scanType: ["qrCode","barCode"], // 可以指定扫二维码还是一维码，默认二者都有
        success: function (res) {
          var strNum = res.resultStr.toString();
          that.fetchProduct(strNum);
        },
        error: function(res){
          if(res.errMsg.indexOf('function_not_exist') > 0){
            that.$dialog.notify({mes: '版本过低，请升级版本！', icon: 'error', timeout: 1200});
          }
        }
      });
    },
    //根据验证码查询物品
    fetchProduct(str) {
      let that = this;
      var opt = {redeemCode:str};
      that.$dialog.loading.open('加载中...');
      that.$http.post(that.apiUrl,opt)
        .then((response) => {
          this.$dialog.loading.close();
          const data = response.data;
          if (data.code === 'S_OK') {
            this.productData = data.var;
            that.tips5 = true;
          }else if(data.code=="REDEEM_CODE_ERROR"){
            that.$dialog.notify({mes: '请输入正确的验证码', icon: 'error', timeout: 2000});
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
