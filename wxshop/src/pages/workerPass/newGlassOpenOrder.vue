<template>
  <section class="service glassOpenOrder glassSupport applyResult glassClaim">
    <article :class="{'s_contract scroll_padding_bottom':isIos,'s_contract':!isIos}">
        <div class="i_top">
          <div class="item s_con">
            <div><span>手机号：</span>
              <input type="text" maxlength="11" v-model="optApply.phone" placeholder="请输入手机号" />
            </div>
          </div>
          <div class="item s_con">
            <div><span>姓名：</span>
                <input type="text" v-model="optApply.customerName" placeholder="请输入姓名" />
            </div>
          </div>
          <div class="item s_con">
            <div>
                <span>车牌号：</span>
                <input type="text" maxlength="11" v-model="optApply.licensePlate" placeholder="请输入车牌号" />
            </div>
          </div>
          <div class="item s_con">
            <div><span>车架号：</span>
              <input type="text" maxlength="17" v-model="optApply.frameNumber" placeholder="请输入车架号" />
            </div>
          </div>
          <div class="item s_con">
            <div><span>金额：</span>
              <input type="text" v-if="isUpdata" maxlength="11" v-model="optApply.insurancePrice" placeholder="请按金额填写" />
              <input type="text" v-else disabled readonly="readonly" maxlength="11" v-model="optApply.insurancePrice" placeholder="请按金额填写" />
            </div>
            <div>元</div>
          </div>
          <div class="item s_con" @click="tiemFocus(1)">
            <div><span>生效时间：</span><div class="datetime-input"><yd-datetime  type="date" ref="datetime1" :start-date="startTime" v-model="documentTime" slot="right"></yd-datetime></div></div>
            <div><i class="p_date">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="tiemFocus(2)">
            <div><span>到期时间：</span><yd-datetime type="date" ref="datetime2" :start-date="optApply.endDate" v-model="optApply.endDate" slot="right"></yd-datetime></div>
            <div><i class="p_date">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="tipOpen(3)">
            <div><span>服务类型：</span><input type="text" v-model="serviceName" disabled readonly="readonly" placeholder="请选择门店" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="tipOpen(1)">
            <div><span>门店：</span><input type="text" v-model="storeName" disabled readonly="readonly" placeholder="请选择门店" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="selectReferrer('referrer')">
            <div><span>推荐人：</span><input type="text" v-model="optApply.salesmanName" disabled readonly="readonly" placeholder="请选择推荐人" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="i_bottom" style="margin-top.29rem;">
          <div class="g_item">
            <div>保险公司系统计费报价单：<br> 
              <div class="file gong">
                <div class="img" v-for="item in imgList.costFile">
                    <img :id="item" v-bind:src="item"/>
                    <!-- <img id="imgContainer" src=""/> -->
                    <div class="img_btn" >
                      <i class="img_big_btn" @click="$refs.child.openTip(item)" v-if="!isWx">&nbsp;</i>
                      <i class="img_big_btn" @click="imgBtn(item)" v-if="isWx">&nbsp;</i>
                      <i class="img_delete_btn" @click="deleteImg(item,'costFile')">&nbsp;</i>
                    </div>
                </div>
                <div class="upImg_gong" v-if="isCostFileShowBtn">
                  <span class="l_up_file" style="margin-top: .2rem;">&nbsp;</span> 
                  <form method="post" enctype="multipart/form-data" id="form" action="upload">
                    <input type="file" id="costFile" @change="readFile('costFile')" accept="image/*"> 
                  </form>
                  <div class="p_20">上传计费报价单</div>
                </div>
              </div>
            </div>
            <div>服务协议：<br> 
              <div class="file gong">
                <div class="img" v-for="item in imgList.agreementFile">
                    <img :id="item" v-bind:src="item"/>
                    <div class="img_btn" >
                      <i class="img_big_btn" @click="$refs.child.openTip(item)" v-if="!isWx">&nbsp;</i>
                      <i class="img_big_btn" @click="imgBtn(item)" v-if="isWx">&nbsp;</i>
                      <i class="img_delete_btn" @click="deleteImg(item,'agreementFile')">&nbsp;</i>
                    </div>
                </div>
                <div class="upImg_gong" v-if="isAgreementFileShowBtn">
                  <span class="l_up_file" style="margin-top: .2rem;">&nbsp;</span>
                  <form method="post" enctype="multipart/form-data" id="form" action="upload">
                    <input type="file" id="agreementFile" @change="readFile('agreementFile')" accept="image/*"> 
                  </form> 
                  <!-- <input type="file" id="agreementFile" @change="readFile('agreementFile')" accept="image/*">  -->
                  <div class="p_20">上传服务协议单</div>
                </div>
              </div>
            </div>
          </div>
        </div>
        </div>
        <div class="btn_glass_save" style="border: 1px solid #ddd;color: #999;" v-if="insuranceStatus==2">已存在生效订单</div>
        <div class="btn_glass_save" @click = "submit('add')" v-else>提交开单</div>
      <!-- 选择门店 -->
      <div class='tip_f' v-if="tips1" v-model="tips1" @click="tipOpen(1)">&nbsp;</div>
      <div class='ck-tipsFormat center-tip service_cancel_tip' v-model="tips1" v-if="tips1">
          <section class="service_cancel">
            <p>申请开通的门店</p>
            <div class="service_li max_height">
                <div :class="{'has': item.isSelect}" v-for="(item,index) in storeList" @click="storeBtn(item,index,'btn')">{{item.orgShortName}}</div>
            </div>
            <div class="bottom_btn fiex_tip">
              <div @click="tipOpen(1,'cancel')">取消</div>
              <div @click="tipOpen(1,'sub')">确定</div>
            </div>
          </section>  
      </div>
      <!-- 选择开单信息 -->
      <div class='tip_f' v-if="tips2" v-model="tips2" @click="tipOpen(2)">&nbsp;</div>
      <div class='ck-tipsFormat center-tip service_cancel_tip' v-model="tips2" v-if="tips2">
          <section class="service_cancel">
            <div class="open_head">请选择开单信息<i class="close_icon" @click="tipOpen(2)">&nbsp;</i></div>
            <div class="open_content max_height">
              <div v-for="(item,index) in openList" @click="openOrderBtn(item,index,'btn')" class="open_li">
                <div>{{item.phone}}</div>
                <div v-if="item.licensePlate">车牌号：{{item.licensePlate}}</div>
                <div v-if="item.frameNumber">车架号：{{item.frameNumber}}</div>
                <div >订单状态：
                  <span v-if="item.insuranceStatus==1">未开始</span>
                  <span v-else-if="item.insuranceStatus==2">进行中</span>
                  <span v-else>已失效</span>
                </div>
              </div>
            </div>
          </section>  
      </div>
      <!--选择当事方信息-->
      <yd-popup v-model="tips3" v-if="tips3" position="bottom" class="ser-legalfield" height="35%">
        <section class="absol">
          <div>
            <span @click="tipClose(3,'cancel')">取消</span>
          </div>
          <div>
            <span @click="tipClose(3,'sub')">确定</span>
          </div>
        </section>  
        <ul class="shop">
          <li @click="selectItem('1')"><span>新购</span><i :class="{'currGou':serviceType=='1','gou':serviceType!='1'}">&nbsp;</i></li>
          <li @click="selectItem('2')"><span>续购</span><i :class="{'currGou':serviceType=='2','gou':serviceType!='2'}">&nbsp;</i></li>
        </ul>
      </yd-popup>
    </article>

    <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
    <yd-imgBig-tip ref="child"></yd-imgBig-tip>
    <div v-if="isFiexShow" class='scroll_bottom_fixed' :style="{'background-color':bStyle}">---我也是有底线滴---</div>
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,isLeapYear,getThisMonthDay,removeSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {citySub} from '../../utils/citySub';
  import {ydFunLib} from '../../utils/ydFunLib';
  import ajaxModel from '../../utils/ajaxModel';
  import {qiniuLib} from '../../utils/qiniuLib';
  import axios from 'axios';
  export default {
    data() {
      return {
        isShowCity:false,
        tips1: false,
        tips2:false,
        tips3:false,
        tips4:false,
        isWx:true,
        isCostFileShowBtn:true,
        isAgreementFileShowBtn:true,
        isUpdata:true,
        imgList: {
          agreementFile:[],
          costFile:[],
        },
        title: '玻璃单独保障服务',
        imgBaseUrl: window.imgBaseUrl,
        apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=insurance&recordSource=2& serviceType=BLBZKD&isBase64=y&flag=thumb',
        apiServerDetailUrl: window.baseUrl+window.appName+'/wxpay/findServerDetail',//服务详情
        apiSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/insurance/wxOpenRecordManage?type=add&openId='+window.openId,//玻璃险开单
        apiSubmitUpdateUrl: window.baseUrl+window.appName+"/shopapi/func/insurance/wxOpenRecordManage?type=update",//玻璃保障修改
        apiGetInfoUrl:window.baseUrl+window.appName+'/shopapi/func/insurance/wxMatchOpenInfo',//获取用户信息
        apiGetFrameNumber:window.baseUrl+window.appName+'/shopapi/func/insurance/wxGetOpenInfoByFrameNumber',//根据车架号查询开单记录
        phoneStatus:'',
        uNameStatus:'',
        datetime2: true,
        storeList:[],
        isFiexShow:false,
        isIos:false,
        bStyle:'#fff',
        storeName:'',
        serviceName:'新购',
        serviceType:'1',
        citySub: {},
        documentTime:'',
        documentEndTime:'',
        startTime:'',
        type:'',
        isApply:false,
        vinStatus:'',
        btnMess:'提交开单',
        insuranceStatus:'3',
        isHasSeecion:false,
        openList:[],
        openToken:'',
        qiniuToken:'',
        optApply:{
          customerName:'',//姓名
          phone:'',//手机号
          licensePlate:'',//车牌号
          frameNumber:'',//车架号
          insurancePrice:'',//服务价格
          startDate:'',//开始时间
          endDate:'',//截止时间
          orgId:'',//门店id
          serviceType:'1',//服务类型
          salesmanId:'',//推荐人id
          costFileId:'',//成本金额ID
          agreementFileId:'',//服务协议ID
          salesmanName:''// 推荐人名称
        }
      }
    },
    watch: {
      '$route.path': 'markInit',//监听当前路由变化（辅助初始化）
       
      'optApply.phone'() { //监听当前输入手机号并实时校验
        if(ydFunLib.telInvalid(this.optApply.phone)) {
          this.phoneStatus = "success";
          this.fetchUserData();
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
      },
      'optApply.frameNumber'() {
        if(!this.vinValid(this.optApply.frameNumber)) {  //监听当前输入车架号并实时校验
          if(ydFunLib.strlenValid(this.optApply.frameNumber)!=17) {
            this.vinStatus = "error";
          }else {
            this.vinStatus = "success";
            if(!this.isHasSeecion&&this.optApply.licensePlate){
              this.getOpenOrder();
            }
          }
        }else {
          this.vinStatus = "error";
        }
      },
      'documentTime'() {
          if(this.type =='' || this.documentTime != this.optApply.startDate){
            var _d = this.documentTime.split('-');
            this.optApply.startDate = this.documentTime;
            this.getEndTime(parseInt(_d[0]),parseInt(_d[1]),parseInt(_d[2]));
          }else{
            this.type ='';
          }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.getNowData();
      this.markInit();
      this.fetchServiceInfo();
      //获取共用token
      ajaxModel.generateToken(this,(result)=>{
        this.openToken = result;
      })
      this.isWx = ydFunLib.curNavigator();
    },
   //添加钩子，监听路由变化时候跳转问题
    beforeRouteLeave(to, from, next){
       var s_1 = getSessionStore('s_1',false);
        if(typeof(s_1) !='boolean'){
          next();
        }else{
          setSessionStore('s_1','111');
          next({path:'/infoPushApply'});
        }
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
      //监听滚动条事件
      scrollChange(){
        var ua = navigator.userAgent;
        this.isIos = !!ua.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/);
        if(this.isIos){
          var scrollTop = document.body.scrollTop;
      　　var wScroll = document.body.scrollHeight;
      　　var wHeight = document.body.offsetHeight;
      　　if(scrollTop + wHeight == wScroll || scrollTop + wHeight>wScroll){
      　　　this.isFiexShow = true;
      　　}else{
            this.isFiexShow = false;
          }
        }
      }, 
      markInit() {
        this.qiniuToken = qiniuLib.genToken();
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        var s_1 = getSessionStore('s_1',false);
        if(typeof(s_1) !='boolean'){
          removeSessionStore('s_1');
        }
        this.type = this.$route.query.type? this.$route.query.type:this.type;
        this.insuranceStatus = this.$route.query.messType&&this.$route.query.messType=='update'?'4':'3';
        var glassOpenOrderSubmitInfo = getSessionStore('glassOpenOrderSubmitInfo',true);
        if(typeof(glassOpenOrderSubmitInfo) !='boolean') {
          this.isHasSeecion = glassOpenOrderSubmitInfo.frameNumber?true:false;
          this.optApply = glassOpenOrderSubmitInfo;
          this.serviceType  = this.optApply.serviceType;
          this.serviceName = this.optApply.serviceType==1?"新购":"续购";
          console.log(this.optApply);
          this.documentTime = this.optApply.startDate.split(' ')[0];
        }else{
          this.getCurDate();
        }

        var glassReferrer = getSessionStore('glass_select_referrer',true);
        if(typeof(glassReferrer) !='boolean') {
          this.optApply.salesmanId = glassReferrer.id.toString();
          this.optApply.salesmanName = glassReferrer.realName;
        }
        var storeData = getSessionStore('glassOpenOrderStoreInfo',true);
        if(typeof(storeData) !='boolean') {
          this.storeList = storeData;
          for(var i=0;i<this.storeList.length;i++){
            if(this.storeList[i].isSelect){
              this.storeName = this.storeList[i].orgShortName;
            }
          }
        }

        var imgData = getSessionStore('glassOpenOrderImgInfo',true);
        if(typeof(imgData) !='boolean') {
          this.imgList = imgData;
          this.isCostFileShowBtn = imgData.costFile.length>0?false:true;
          this.isAgreementFileShowBtn = imgData.agreementFile.length>0?false:true;
        }
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_GLASSOPENORDER});
      },
      //我要修改
      hasUpdate(){
        this.isUpdata = true;
        this.insuranceStatus = '';
        this.type=2;
        console.log(this.documentTime,'ddddd');
      },
      vinValid(value,allowEmptyString) { //校验车架号方法
        let val= value.replace(/\s/g,"");
        return (val === null) || (val === undefined)
          || (!allowEmptyString ? val === '' : false)
          || (val.length === 0);
      },
      //开单信息查找
      openOrderBtn(item){
        this.optApply.frameNumber = item.frameNumber?item.frameNumber:'';
        this.optApply.licensePlate = item.licensePlate;
        this.optApply.customerName = item.name;
        this.optApply.phone = item.phone;
        this.insuranceStatus = item.insuranceStatus;
        this.tips2 = false;
      },
      //根据车架号查询开单记录
      getOpenOrder(){
        let that = this;
        var _opt = {
          "frameNumber":that.optApply.frameNumber,
          "licensePlate":that.optApply.licensePlate,
          "insuranceStatus":that.insuranceStatus
        };
        that.$http.post(that.apiGetFrameNumber,_opt)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(data.var){
                that.type = 2;
                that.documentTime = data.var.startDateString;
                that.optApply =  ydFunLib.ydExtend(that.optApply,data.var);
                that.optApply.startDate = that.documentTime;
                that.optApply.endDate = data.var.endDateString;
                that.insuranceStatus = data.var.insuranceStatus;
                that.imgList.costFile = [];
                that.imgList.agreementFile = [];
                that.imgList.costFile.push(that.imgBaseUrl+data.var.costFileUrl);
                if(data.var.agreementFileId){
                  that.imgList.agreementFile.push(that.imgBaseUrl+data.var.agreementFileUrl);
                  that.isAgreementFileShowBtn=false;
                }
                that.isCostFileShowBtn=false;
                that.isShowBtn = false;
                if(that.insuranceStatus==1||that.insuranceStatus==2){
                  that.isUpdata = false;
                }
                console.log(that.optApply,'that.optApply',that.documentTime);
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
      imgBtn(pId){
        ydFunLib.wxBigImg(this.imgBaseUrl+'/shop/downLoad/showLogo/'+pId);
      },
      //选择服务类型
      selectItem(str){
        this.serviceType = str;
      },
      //删除图片-车辆成本
      deleteImg(fileId,str){
        var _len = this.imgList[str];
        var len = _len.filter(function(data){
          return
        });
        for(var i=0;i<_len.length;i++){
          if(fileId == _len[i]){
            _len.splice(i,1);
            break;
          }
        }
        this.imgList[str] = _len;
        this.optApply[str+'Id'] = '';
        if(str == 'costFile'){
          this.isCostFileShowBtn = true;
        }
        if(str == 'agreementFile') {
          this.isAgreementFileShowBtn = true;
        }
      },
      tiemFocus(n){
        this.$refs['datetime'+n].open();
      },
      tipClose(n,pT){
        if(pT=='sub'){
          this.optApply.serviceType = this.serviceType;
          this.serviceName = this.serviceType==1?"新购":"续购";
        }else{
          this.serviceName = this.optApply.serviceType ==1?"新购":"续购";
        }
        this['tips'+n] = !this['tips'+n];
      },
      tipOpen(n,str) {
        this['tips'+n] = !this['tips'+n];
        if(str=='sub'){
          var len = this.storeList.filter(function(data){
            return data.isSelect == true;
          })
          if(len.length==0){
            this.$dialog.notify({mes: '请选择门店', icon: 'error', timeout: 1200});
            return;
          }
        }
      },
      //门店点击
      storeBtn(opt,pIndex,str) {
        var curOpt = this.storeList[pIndex];
        this.storeList[pIndex].isSelect = !curOpt.isSelect;
        this.storeList.filter(function(data){
          if(data.orgId != opt.orgId){
            data.isSelect = false;
            return data;
          }
        })
        if(this.storeList[pIndex].isSelect) {
          this.optApply.orgId = curOpt.orgId;
          this.storeName = curOpt.orgShortName;
          this.optApply.salesmanId='',//推荐人id
          this.optApply.salesmanName=''// 推荐人名称
          }else{
            this.optApply.orgId = '';
            this.storeName = '';
            this.optApply.salesmanId='',//推荐人id
            this.optApply.salesmanName=''// 推荐人名称
          }
        removeSessionStore('glass_select_referrer');
      },
      removeSession(){
        removeSessionStore('glass_select_referrer');
        removeSessionStore('glassOpenOrderSubmitInfo');
        removeSessionStore("glassOpenOrderStoreInfo");
        removeSessionStore("glassOpenOrderImgInfo");
      },
      glassOpenOrderInfo() {
        //保存用户输入信息
        this.optApply.startDate = this.documentTime;
        setSessionStore("glassOpenOrderSubmitInfo", this.optApply);
        setSessionStore("glassOpenOrderStoreInfo", this.storeList);
        setSessionStore("glassOpenOrderImgInfo", this.imgList);
        if(this.optApply.salesmanId){
          var opt = {
            id : this.optApply.salesmanId,
            realName:this.optApply.salesmanName
          }
          setSessionStore('glass_select_referrer',opt);
        }
        
      },
      //跳去选择推荐人、门店
      selectReferrer(str) {
        this.glassOpenOrderInfo();
        setSessionStore('s_1',str);
        //跳去地址
        this.$router.push({
        path: '/'+str,
          query: {
            from: 'glassOpenOrder',
            orgId: this.optApply.orgId,
            messType: this.insuranceStatus==3?'add':'update'
          }
        });
      },
      getNowData(){
        var curDate = new Date();
        var y = curDate.getFullYear();
        var m = curDate.getMonth()+1>9?curDate.getMonth()+1:'0'+ (curDate.getMonth()+1);
        var d = curDate.getUTCDate()>9?curDate.getUTCDate():'0'+ curDate.getUTCDate();
        this.startTime= y+'-'+m+'-'+d;
      },
      //获取当前时间
      getCurDate(){
        var curDate = new Date();
        var y = curDate.getFullYear();
        var m = curDate.getMonth()+1>9?curDate.getMonth()+1:'0'+ (curDate.getMonth()+1);
        var d = curDate.getUTCDate()>9?curDate.getUTCDate():'0'+ curDate.getUTCDate();
        this.documentTime = y+'-'+m+'-'+d;
        this.startTime= this.documentTime;
        this.getEndTime(parseInt(y),parseInt(m),parseInt(d));
      },
      //获取结束时间
      getEndTime(y,m,d){
        if(d==1){//某月的1号
          if(m==1){//当前月份是1月
            this.optApply.endDate = y +'-12-31';
          }else{
            var cd = getThisMonthDay(y+1,m-1);
            var _cm = parseInt(m-1)>9?parseInt(m-1):'0'+parseInt(m-1);
            var _cd = parseInt(cd)>9?parseInt(cd):'0'+(parseInt(cd));
            this.optApply.endDate = (y+1)+'-'+_cm+'-'+_cd;
            // this.optApply.endDate = (y+1)+'-'+(m-1)+'-'+cd;
          }
        }else{
          var _cm = parseInt(m)>9?parseInt(m):'0'+parseInt(m);
          var _cd = parseInt(d)-1>9?parseInt(d)-1:'0'+(parseInt(d)-1);
          this.optApply.endDate = (parseInt(y)+1)+'-'+_cm+'-'+_cd;
        }
      },
      fetchServiceInfo(){
        //获取服务详情
        let that = this;
        var opt ={
          applyId : this.$route.query.applyId,
          serverCode: this.$route.query.serverCode
        };
        $('.yd-datetime-input').removeClass("yd-datetime-input").addClass('datetime-input');
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiServerDetailUrl,opt)
          .then((response) => {
            const data = response.data;
            if (data.code === 'S_OK') {
              this.storeList = data.var.orgLists;
              that.optApply.orgId = that.optApply.orgId==''?data.var.orgId:that.optApply.orgId;
              for(var i=0;i<this.storeList.length;i++){
                if(that.optApply.orgId == that.storeList[i].orgId){
                  that.storeList[i].isSelect = true;
                  that.storeName = that.storeList[i].orgShortName;
                }else{
                  this.storeList[i].isSelect = false;
                }
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
      submit(str){
        let that = this;
        //提交数据
        if(that.optApply.phone == ''){
          that.$dialog.notify({mes: '请输入手机号码', icon: 'error', timeout: 1200});
          return;
        }else if(this.phoneStatus == "error"){
          that.$dialog.notify({mes: '请输入正确的手机号', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.customerName == ''){
          that.$dialog.notify({mes: '请输入姓名', icon: 'error', timeout: 1200});
          return;
        }else if(this.uNameStatus == "error"){
          that.$dialog.notify({mes: '请输入正确的姓名', icon: 'error', timeout: 1200});
          return;
        }
        
        //校验输入的车架号
        if(that.vinStatus != "success") {
          if(ydFunLib.strlenValid(that.optApply.frameNumber) != 17) {
            that.$dialog.notify({mes: '输入17位数车架号', icon: 'error', timeout: 1200});
          }else if(!ydFunLib.isNumberLetters(that.optApply.vin)) {
            that.$dialog.notify({mes: '请输入您的车架号格式不正确', icon: 'error', timeout: 1200});
          }else {
            that.$dialog.notify({mes: '请输入您的车架号', icon: 'error', timeout: 1200});
          }
          return
        }
        if(that.optApply.insurancePrice == ''){
          that.$dialog.notify({mes: '请输入服务价格', icon: 'error', timeout: 1200});
          return;
        } 
        if(that.optApply.orgId == ''){
          that.$dialog.notify({mes: '请选择门店', icon: 'error', timeout: 1200});
          return;
        } 
        if(that.optApply.salesmanName == ''){
          that.$dialog.notify({mes: '请选择推荐人', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.costFileId == ''){
          that.$dialog.notify({mes: '上传本车辆成本金额照片', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.agreementFileId == ''){
          that.$dialog.notify({mes: '上传服务协议照片', icon: 'error', timeout: 1200});
          return;
        }
        console.log('111',this.optApply);
        if(this.isApply){
          return false;
        }else{
          this.isApply = true;
        }
        console.log('11333',this.optApply);
        this.$dialog.loading.open('保存中...');
        this.optApply.startDate = this.documentTime;
        this.glassOpenOrderInfo();
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_GLASSOPENORDER_BTN_SAVE});
        var _url = str=='add'?this.apiSubmitUrl+'&token='+this.openToken:this.apiSubmitUpdateUrl;
        this.$http.post(_url,this.optApply)
        .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
              that.removeSession();
              that.goTO();
            }});
            that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_GLASSOPENORDER_MESS_SAVE_OK});
          }else {
            this.isApply = false;
            that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_GLASSOPENORDER_MESS_SAVE_FAIL});
            if(data.code=='PARAM_ISNULL' && data.msg ==''){
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }else if(data.code=="ALREADY_EXIST_DATA"){
              that.$dialog.notify({mes: '已存在车架号相同的开单记录', icon: 'error', timeout: 2500});
            }else if (data.code == "NOT_ALLOW_UPDATE"){
              that.$dialog.notify({mes: '已效期或已过期记录不可修改', icon: 'error', timeout: 2500});
            }else if(data.code=="NOT_ALLOW_UPDATE_TODAY"){
              that.$dialog.notify({mes: ' 只允许修改今天新增的开单记录', icon: 'error', timeout: 2500});
            }else if (data.code == "PARAM_ERROR"){
              that.$dialog.notify({mes: '开单记录不存在', icon: 'error', timeout: 2500});
            }else if (data.code == "NO_OPEN_SERVICE"){
              that.$dialog.notify({mes: '没有开通服务', icon: 'error', timeout: 2500});
            }else if (data.code == "OPENID_ISNULL"){
              that.$dialog.notify({mes: '当前会话已失效，请重新进入或刷新当前页面!', icon: 'error', timeout: 2500});
            }else{
              that.$dialog.notify({mes: "操作失败,请稍后再试", icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          setTimeout(()=>{
            this.$dialog.loading.close();
            this.isApply = false;
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          },300);
        });
      },
      goTO() {
        this.$router.push({
          path: '/infoPushApply'
        });
      },
      //获取用户信息
      fetchUserData(){
        let that = this;
        var glassOpenOrderSubmitInfo = getSessionStore('glassOpenOrderSubmitInfo',true);
        if(typeof(glassOpenOrderSubmitInfo) !='boolean' && glassOpenOrderSubmitInfo.phone == that.optApply.phone) {
          return false;
        }
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiGetInfoUrl,{"phone":that.optApply.phone})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(data.var && data.var.isData == 1){
                that.openList = data.var.matchList;
                if(data.var.matchList.length==1){
                  var opt = data.var.matchList[0];
                  that.optApply.frameNumber = opt.frameNumber?opt.frameNumber:'';
                  that.optApply.licensePlate = opt.licensePlate?opt.licensePlate:'';
                  that.optApply.customerName = opt.name;
                  that.insuranceStatus = opt.insuranceStatus?opt.insuranceStatus:that.insuranceStatus;
                  that.optApply.phone = opt.phone;
                }else{
                  that.tips2 = true;
                }
                console.log('that.openList',that.openList,data.var.matchList);
              }
            }
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
       //获取本地图片信息
      readFile(str){ 
        var that = this;
        var file = document.getElementById(str).files[0]; //获取file对象
        //判断file的类型是不是图片类型。
        if(!/image\/\w+/.test(file.type)){ 
            alert("文件必须为图片！"); 
            return false; 
        }
        if(file){
          qiniuLib.uploadWithSDK(file,(key,domain)=>{
            if(str == 'agreementFile'){
              that.optApply.agreementFileId = domain+key;
              that.imgList.agreementFile.push(domain+key);
              that.isAgreementFileShowBtn=false;
            }else if(str == 'costFile'){
              that.optApply.costFileId = domain+key;
              that.imgList.costFile.push(domain+key);
              that.isCostFileShowBtn=false;
            }
          });
        }
      },
       
      //上传图片
      prodPicUpload(base64,str){
        var that = this;
        // this.$dialog.loading.open('上传中...');
        this.tips4 = true;
        axios.post(this.apiUploadFileUrl, JSON.stringify({"imgStr":base64}),{headers: {'Content-Type': 'application/xml'}}).then(function (response) {
               const data = response.data;
            if(data.code == "S_OK") {
              var fileId = data["var"];
                  if(fileId){
                    if(str == 'agreementFile'){
                      that.optApply.agreementFileId = fileId;
                      that.imgList.agreementFile.push(fileId);
                      that.isAgreementFileShowBtn=false;
                    }else if(str == 'costFile'){
                      that.optApply.costFileId = fileId;
                      that.imgList.costFile.push(fileId);
                      that.isCostFileShowBtn=false;
                    }
                  }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
              
          }).catch(function (error) {
            that.$dialog.loading.close();
            that.tips4 = false;
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      }
    }
  }
  
 
</script>

