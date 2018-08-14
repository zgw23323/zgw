<template>
  <section class="service applyResult" style="background-color: #f5f5f5;">
    <article class="s_contract glassSupport">
        <div class="glassSupport_statu">
          <p>预约成功</p>
          <p>请在预定的时间准时到达噢~</p>
        </div>
        <div class="i_top">
          <div class="item s_con">
            <div>
              <span>姓名：</span>
              <input type="text" v-if="isUpdata" @focus="handleFocus" v-on:blur="changeInput" maxlength="20" v-model="optApply.customerName" placeholder="请输入姓名" />
              <input type="text" v-else disabled readonly="readonly" v-model="optApply.customerName" />
            </div>
          </div>
          <div class="item s_con">
            <div>
              <span>手机号：</span>
              <input type="number" v-if="isUpdata" @focus="handleFocus" v-on:blur="changeInput" maxlength="11" v-model="optApply.phone" placeholder="请输入手机号" />
              <input type="text" disabled v-else readonly="readonly" v-model="optApply.phone" />
            </div>
          </div>
          <div class="item s_con" v-if="isUpdata">
            <div><span>车牌号/车架号：</span><input type="text" readonly="readonly" v-model="optApply.frameNumber" placeholder="请输入车牌号/车架号" />
            </div>
           <div @click="tipOpen(3)"><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con" v-else>
            <div><span>车牌号/车架号：</span><input type="text" v-model="optApply.frameNumber" disabled readonly="readonly" />
            </div>
          </div>
          <div class="item no_bor s_con" v-if="isUpdata" @click="serectCart('carList')">
            <div><span>车系：</span><input type="text" v-model="optApply.carName" readonly="readonly" placeholder="请选择预约的车系" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item no_bor s_con" v-else>
            <div><span>车系：</span><input type="text" v-model="optApply.carName" disabled readonly="readonly" />
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item s_con" style="width: 92%;">
            <div class="i_item_time"><span>预约时间：</span><input type="text" v-if="isUpdata" readonly="readonly" v-model='nowSelectData'  @focus="tiemFocus" placeholder="请选择时间" /><input type="text" v-else disabled readonly="readonly" v-model='nowSelectData' /></div>
            <div v-if="isUpdata" @click="tiemFocus"><i class="p_date">&nbsp;</i></div>
          </div>
          <div class="item no_bor s_con" v-if="isUpdata" @click="serectCart('serviceStore')">
            <div><span>预约4s店：</span><input type="text" v-model="storeInfo.orgShortName" readonly="readonly" placeholder="请选择您预约的4s店" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item no_bor s_con" v-else>
            <div><span>预约4s店：</span><input type="text" v-model="storeInfo.orgShortName" disabled readonly="readonly" />
            </div>
          </div>
          <div class="item no_bor s_con" style="padding: 0 0 .51rem;" v-if="storeInfo.orgShortName != ''">
            <div class="service_store_mess">
              <p>店称：<span class="span1">{{storeInfo.orgShortName}}</span></p>
              <p>电话：<span class="span1">{{storeInfo.mobilePhone}}</span></p>
              <p><span>地址：</span><span style="width:85%;" class="span1">{{storeInfo.address}}</span></p>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor s_con s_textarea">
            <div>维修部位<span class="t_mess">（确认真实维修部位，以便4S店为您准备）</span></br>
              <ul class="pepair_parts" style="margin-bottom: 0;">
                <li :class="{'curBtn':item.isSelect,'noBtn':!item.isSelect && !isUpdata}" v-for="item in positionData" @click="positionBtn(item)"><span>{{item.name}}</span></li>
              </ul>
            </div>
          </div>
        </div>
        <div class="i_bottom" v-if="firstPosition">
          <div class="item no_bor s_con">
            <div>上传照片<span class="t_mess">（可选项）</span></br>
              <div class="file_mess">
                  <p>注意：1、拍摄照片时,对维修玻璃需从3个不同角度进行拍摄；</p>
                  <p style="margin-left:.72rem;margin-top:.13rem;">2、照片中需要有维修车辆车牌号码；</p>
              </div>
              <ul class="img_demo">
              <li v-for="item in positionData" v-if="item.isSelect && item.repairCode==firstPosition" :class="'img_'+item.repairCode">&nbsp;</li>
              </ul>
              <div v-for="(item,index) in positionData" v-if="item.isSelect" class="glassSupport_flex">
                <div :class="{'upImg no_bor':imgData.picId !='','upImg': imgData.picId ==''}" v-for="imgData in item.picIds">
                  <!-- <form method="post" enctype="multipart/form-data" id="form" action="upload"> -->
                    <!-- <input type="file" v-if="isUpdata" id="demo1"  capture="camera" accept="image/*"/> -->
                  <!-- </form> -->
                  <input type="file" v-if="isUpdata" :id="item.repairCode+'_'+imgData.visualType" @change="readFile(index,item.repairCode+'_'+imgData.visualType)" capture="camera" accept="image/*"/>
                  <div v-if="imgData.picId!=''" class="img_look">
                    <img :id="imgData.picId" v-bind:src="imgData.base64"/>
                    <div class="img_btn">
                      <i class="img_big_btn" @click="$refs.child.openTip(imgData.base64)" v-if="!isWx">&nbsp;</i>
                      <i class="img_big_btn" @click="imgBtn(imgData.base64)" v-if="isWx">&nbsp;</i>
                      <i class="img_delete_btn" @click="imgDelete(index,imgData)">&nbsp;</i>
                    </div>
                  </div>
                  <div v-else>
                    <span>{{item.name}}</span>
                    <div class="p_20">{{imgData.message}}</div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="i_bottom" style="margin-bottom: 1.55rem;">
          <div class="item no_bor s_con s_textarea">
            <div>备注：</br>
            <textarea v-if="isUpdata" @focus="handleFocus" maxlength="200" v-on:blur="changeInput" v-model="optApply.remark" placeholder=""></textarea>
            <textarea v-else readonly="readonly" maxlength="200" v-model="optApply.remark" placeholder=""></textarea>
            </div>

          </div>
        </div>
        
        <div :class="{'glassSupport_btn dis':isKeyUp,'glassSupport_btn':!isKeyUp}">
          <div v-if="isUpdata" @click = "submit" class="g_btn1">保存预约信息</div>
          <div v-else @click = "isUpdata = true" class="g_btn1">修改预约信息</div>
          <div class="g_btn2" @click="tips1=true;">取消预约</div>
        </div>
    </article>
    
    <!--选择服务时间-->
    <yd-popup v-model="tips2" position="center" class="ser-legalfield" height="80%" width="92%">
        <section class="glassSupport_tip">
            <div class="tip_top">
              <div class="tip_title_mess">预约时间</div>
              <div class="tip_close_icon" @click="closeTip(2)">&nbsp;</div>
            </div>
            <div class="tip_content" style="padding:.47rem .28rem 0;">
                
                <div class="select_year" v-if="isShowYear">
                  <p>年份</p>
                  <div id='year' class="year"><span :class="{'curBtn':item==selectYear}" v-for="(item,index) in yearList" @click="yearBtn(item)">{{item}}年</span></div>
                </div>
                <div class="select_date">
                  <article class="date">
                    <div class="month" @click='isshowMonth=!isshowMonth;' v-if="isShowYear"><span id="month">{{selectMonth}}月</span><i class="up_month">&nbsp;</i></div>
                    <div class="month"><span id="month" v-if="!isShowYear">{{curMonth}}月</span></div>
                    <!-- <div class="month" @click='isshowMonth=!isshowMonth;'><span id="month">10月</span></div> -->
                    <div class="month_select" v-if="isshowMonth">
                      <span>1月</span>
                      <span>2月</span>
                      <span>3月</span>
                      <span>4月</span>
                      <span>5月</span>
                      <span>6月</span>
                      <span>7月</span>
                      <span>8月</span>
                      <span>9月</span>
                      <span>10月</span>
                      <span>11月</span>
                      <span>12月</span>
                    </div>
                    <div id='day' class="day"><span :class="{'no_btn d_item':index==0,'cur d_item':selectYear==item.yearValue && selectMonth ==item.monthValue && selectDay == item.dayValue,'d_item':item.statu=='3'||item.statu=='1'}" v-for="(item,index) in dayList" :id="item.monthValue+'_'+item.dayValue" @click="dayBtn(item,index)">{{item.dayValue}}</span></div>
                  </article>
                  <i class="r_shadow">&nbsp;</i>             
                </div>
                <div class="select_hour">
                  <div class="item_1">
                    <p>上午</p>
                    <ul>
                      <li class="noBtn" v-if="isNowDay && (nowHour==10 && nowMinutes>0 || nowHour>10)">9:00-10:00</li>
                      <li :class="{'curBtn':selectHour=='9'}" @click="hourBtn('9')" v-else>9:00-10:00</li>
                      <li class="noBtn" v-if="isNowDay && (nowHour==11 && nowMinutes>0 || nowHour>11)">10:00-11:00</li>
                      <li :class="{'curBtn':selectHour=='10'}" v-else @click="hourBtn('10')">10:00-11:00</li>
                      <li class="noBtn" v-if="isNowDay && (nowHour==12 && nowMinutes>0 || nowHour>12)">11:00-12:00</li>
                      <li :class="{'curBtn':selectHour=='11'}" v-else @click="hourBtn('11')">11:00-12:00</li>
                    </ul>
                  </div>
                  <div class="item_2">
                    <p>下午</p>
                    <ul>
                      <li class="noBtn" v-if="isNowDay && (nowHour==15 && nowMinutes>0 || nowHour>15)">14:00-15:00</li>
                      <li :class="{'curBtn':selectHour=='14'}" v-else @click="hourBtn('14')">14:00-15:00</li>
                      <li class="noBtn" v-if="isNowDay && (nowHour==16 && nowMinutes>0 || nowHour>16)">15:00-16:00</li>
                      <li :class="{'curBtn':selectHour=='15'}" v-else @click="hourBtn('15')">15:00-16:00</li>
                      <li class="noBtn" v-if="isNowDay && (nowHour==17 && nowMinutes>0 || nowHour>17)">16:00-17:00</li>
                      <li :class="{'curBtn':selectHour=='16'}" v-else @click="hourBtn('16')">16:00-17:00</li>
                      <li class="noBtn" v-if="isNowDay && (nowHour==18 && nowMinutes>0 || nowHour>18)">17:00-18:00</li>
                      <li :class="{'curBtn':selectHour=='17'}" v-else @click="hourBtn('17')">17:00-18:00</li>
                    </ul>
                  </div>
                </div>
            </div>
            <div class="tip_bottom">
              <div class="tip_btn_mess" @click="submitData(true)">确定</div>
            </div>
        </section>
    </yd-popup>

    <!--取消预约弹窗-->
    <!-- <yd-popup v-model="tips1" position="center" class="service_cancel_tip" width="92%"> -->
    <div class='tip_f' v-show="tips1" v-model="tips1" @click="tipOpen(3)">&nbsp;</div>
      <div class='ck-tipsFormat center-tip service_cancel_tip' v-model="tips1" v-show="tips1">
        <article class="service_cancel" style="height: 2.6rem;">
            <p style="margin: 1rem 0;font-size:.36rem;color:#000;">确定取消玻璃维修预约吗？</p>
            <div class="bottom_btn">
              <div @click="tips1=false;">取消</div>
              <div @click="submitCancelApply">确定</div>
            </div>
        </article> 
    </div>
    <!-- </yd-popup> -->

      <div class='tip_f' v-show="tips3" v-model="tips3" @click="tipOpen(3)">&nbsp;</div>
      <div class='ck-tipsFormat center-tip service_cancel_tip' v-model="tips3" v-show="tips3">
          <section class="service_cancel">
            <p>请选择预约车牌号</p>
            <div class="service_li max_height">
                <div :class="{'has': item==optApply.frameNumber}" v-for="item in plateNumberList" @click="carNumBtn(item)">{{item}}</div>
            </div>
            <div class="bottom_btn fiex_tip">
              <div @click="tipOpen(3)">取消</div>
              <div @click="tipOpen(3)">确定</div>
            </div>
          </section>  
      </div>
      <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
      <yd-imgBig-tip ref="child"></yd-imgBig-tip>
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,ydExtend,removeSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {citySub} from '../../utils/citySub';
  import {ydFunLib} from '../../utils/ydFunLib';
  import ajaxModel from '../../utils/ajaxModel';
  import {qiniuLib} from '../../utils/qiniuLib';
  export default {
    data() {
      return {
        isWx:true,
        tips1: false,
        tips2:false,
        tips3:false,
        tips4:false,
        winHeight:0,
        btnShow:true,
        title: '申请玻璃保修服务',
        isshowMonth:false,
        isKeyUp:false,
        nowSelectData:'',
        isUpdata:false,
        plateNumberList:[],
        // isSelectNum:'',
        imgBaseUrl: window.imgBaseUrl,
        apiApplyRecordManage: window.baseUrl+window.appName+'/shopapi/func/insurance/wxApplyRecordManage?shopsid='+ window.shopsid +'&type=view',//获取用户预约记录信息
        apiSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/insurance/wxApplyRecordManage?shopsid='+window.shopsid +'&type=update',//保存
        apiCancelReserv: window.baseUrl+window.appName+'/shopapi/func/insurance/wxCancelReservat?shopsid='+window.shopsid,//取消预约
        apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=glass&isBase64=y&flag=thumb',
        phoneStatus:'',
        uNameStatus:'',
        positionData:[
          {
            name:'前挡风玻璃',
            value:'1',
            isSelect:false,
            repairCode:'qiandangfengGlass',
            picIds:[
              {visualType:"1",picId:"",message:"拍摄左视角"},
              {visualType:"2",picId:"",message:"拍摄中视角"},
              {visualType:"3",picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'后挡风玻璃',
            value:'2',
            isSelect:false,
            repairCode: 'houdangfengGlass',
            picIds:[
              {visualType:"1",picId:"",message:"拍摄左视角"},
              {visualType:"2",picId:"",message:"拍摄中视角"},
              {visualType:"3",picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右前门玻璃',
            value:'3',
            isSelect:false,
            repairCode: 'zuoyouqianmenGlass',
            picIds:[
              {visualType:"1",picId:"",message:"拍摄左视角"},
              {visualType:"2",picId:"",message:"拍摄中视角"},
              {visualType:"3",picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右中门玻璃',
            value:'4',
            isSelect:false,
            repairCode: 'zuoyouzhongmenGlass',
            picIds:[
              {visualType:"1",picId:"",message:"拍摄左视角"},
              {visualType:"2",picId:"",message:"拍摄中视角"},
              {visualType:"3",picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右尾侧玻璃',
            value:'5',
            isSelect:false,
            repairCode: 'zuoyouweiceGlass',
            picIds:[
              {visualType:"1",picId:"",message:"拍摄左视角"},
              {visualType:"2",picId:"",message:"拍摄中视角"},
              {visualType:"3",picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右小三角玻璃',
            value:'6',
            isSelect:false,
            repairCode: 'zuoyouxiaosanjiaoGlass',
            picIds:[
              {visualType:"1",picId:"",message:"拍摄左视角"},
              {visualType:"2",picId:"",message:"拍摄中视角"},
              {visualType:"3",picId:"",message:"拍摄右视角"}
            ]
          }
        ],
        selectYear:'',
        selectMonth:'',
        selectDay:'',
        selectHour:'9',
        firstPosition:'',
        nowHour:0,
        isNowDay: true,
        nowMinutes:0,
        isShowYear:false,
        yearList:[],
        dayList:[],
        citySub:{},//城市简称
        storeInfo:{
          orgShortName:'',
          mobilePhone:'',
          address:''
        },//选择的门店信息
        optCancelReserv: {
          id:''
        },
        curMonth:'',
        qiniuToken:'',
        optApply:{
          recordType:'2',//默认
          customerName:'',//姓名
          phone:'',//手机号
          licensePlate:'',//车牌号
          frameNumber:'',//车架号
          orgId:'',//门店
          id:'',
          carId: '',//车系ID
          carName: '',//车系名称
          appointedStartTime:'',// 预约开始时间
          appointedEndTime:'',//预约结束时间
          picArr:[],//维修部位照片组合
          remark:''//备注
        },
        appLyDate:{}//用户预约记录数据
      }
    },
    watch: {
      '$route.path': 'markInit',//监听当前路由变化（辅助初始化）
      'optApply.phone'() { //监听当前输入手机号并实时校验
        if(ydFunLib.telInvalid(this.optApply.phone)) {
          this.phoneStatus = "success";
          this.optApply.phone = this.optApply.phone.substring(0,11);
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
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      document.getElementById("day").addEventListener('scroll',this.winScroll,false);
      //设置年份
      this.setYear();
      this.setDay(12);//设置天数
      this.setTime();//设置可选时间
      if(this.$route.query.type=='1'){
        this.fetchApplyRecordDate();
      }

      let that = this;
      ajaxModel.generateToken(this,(result)=>{
        that.qiniuToken = result.qiniuToken;
        // qiniuLib.uploadWithSDK(result.qiniuToken,'demo1');
      },'?tokenType=qiniu');
    },
    //添加钩子，监听路由变化时候跳转问题
    beforeRouteLeave(to, from, next){
       var s_1 = getSessionStore('s_1',false);
        if(typeof(s_1) !='boolean'){
          next();
        }else{
          setSessionStore('s_1','111');
          next({path:'/serviceHome'});
        }
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
     
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.winHeight = document.documentElement.clientHeight;
        var s_1 = getSessionStore('s_1',false);
        if(typeof(s_1) !='boolean'){
          removeSessionStore('s_1');
        }
        this.getSessionFun();
        //设置当前编辑状态
        this.isUpdata = this.$route.query.type=='1'?false:true;
        //判断当前系统是否是微信浏览器
        this.isWx = ydFunLib.curNavigator();
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_CONTRACT});
      },
      tipOpen(tip) {
        this['tips'+tip] = !this['tips'+tip];
      },
      carNumBtn(str){
        // this.isSelectNum = str;
        this.optApply.frameNumber = str;
      },
      imgBtn(pId){
        ydFunLib.wxBigImg(this.imgBaseUrl+'/shop/downLoad/showLogo/'+pId);
      },
      //维修部位选择
      positionBtn(opt){
        if(this.isUpdata) {
          this.positionData[parseInt(opt.value)-1].isSelect = !this.positionData[parseInt(opt.value)-1].isSelect;
          this.setImgDataList();
        }
      },
      //选择维修部位后展示相应部位照片上传控件
      setImgDataList() {
        var len = [];
        for(var i=0;i<this.positionData.length;i++){
          var item = this.positionData[i];
          if(item.isSelect){
            var opt = {};
            opt.name = item.name;
            opt.value = item.value;
          }
        }
        var len= this.positionData.filter(function(data){
          return data.isSelect;
        });
        if(len.length>0){
          this.firstPosition = len[0].repairCode;
        }
      },
       
      //选择车型
      serectCart(str) {
          this.setSubmitDate();
          this.setSessionFun();
          setSessionStore('s_1',str);
          this.$router.push({
          path: '/'+str,
            query: {
              from: 'glassSupportEdit',
              id:this.$route.query.id,
              code: this.$route.query.code
            }
          });
      },
      setSessionFun() {
        //缓存选择的维修部位
        setSessionStore('glassSupportPostion',this.positionData);
        //缓存填写的信息
        setSessionStore('glassSupportInfo',this.optApply);
        //缓存选择的门店信息
        setSessionStore('serviceStore_btn_info',this.storeInfo);
      },
      getSessionFun() {
        //拿到缓存的用户基本信息
        var glassSupport = getSessionStore('glassSupportInfo',true);
        if(typeof(glassSupport) !='boolean') {
          this.optApply = glassSupport;
        }
        //拿到用户选择的维修部位信息
        var positionData = getSessionStore('glassSupportPostion',true);
        if(typeof(positionData) !='boolean'){
          this.positionData = positionData;
          var len= this.positionData.filter(function(data){
            return data.isSelect;
          });
          if(len.length>0){
            this.firstPosition = len[0].repairCode;
          }
        }
        //拿到用户选择的车系信息 
        var carOpt = getSessionStore('carName',true);
        if(typeof(carOpt) !='boolean'){
          this.optApply.carId = carOpt.carsId;
          this.optApply.carName = carOpt.carsName;
        }
        //拿到用户选择的门店信息
        var storeData = getSessionStore('serviceStore_btn_info',true);
        if(typeof(storeData) != 'boolean'){
          this.storeInfo = storeData;
          this.optApply.orgId = storeData.id;
        }
        //从缓存中拿到车架列表
        var plateNumberList = getSessionStore('plateNumberList',true);
        if(typeof(plateNumberList) !='boolean') {
           this.plateNumberList = plateNumberList;
        }else{
          this.plateNumberList = glassSupport.openFrameNumberList;
        }
      },
      closeTip(tips){
        this['tips'+tips] = false;
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
      tiemFocus(){
        this.goToScrollLeft();
        this.tips2 = true;
      },
      tipClose(tips,str){
        this['tips'+tips]=false;
        if(str){
          this.optApply[str] = this[str];
        }
      },
      openTip(tips){
        this['tips'+tips] = true;
      },
      //选择类型项
      selectItem(pStr,pItem){
        this[pStr] = pItem;
      },
      //获取预约信息
      fetchApplyRecordDate(){
        var that = this;
        that.$http.post(this.apiApplyRecordManage,{id:this.$route.query.id})
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(this.$route.query.type=='1'){
                that.optCancelReserv.id = data.var.id;
                that.optApply = ydExtend(that.optApply,data["var"],false);
                var _time = data.var.appointedEndTimeString.split(' ')[0].split('-');
                that.selectYear = _time[0];
                that.selectMonth = parseInt(_time[1]);
                that.selectDay = parseInt(_time[2]);
                that.selectHour = parseInt(data.var.appointedStartTimeString.split(' ')[1].split(':')[0]);
                that.submitData(false);
                that.storeInfo.orgShortName = data.var.orgName;
                that.storeInfo.mobilePhone = data.var.mobilePhone;
                that.storeInfo.address = data.var.address;
                that.storeInfo.id = data.var.orgId;
                for(var key in data.var.fileMap){
                  for(var i=0;i<that.positionData.length;i++){
                    var _i = that.positionData[i];
                    if(key==_i.repairCode){
                      that.positionData[i].isSelect = true;
                      for(var j=0;j<data.var.fileMap[key].length;j++){
                        var _j = data.var.fileMap[key][j];
                        that.positionData[i].picIds[parseInt(_j.visualType)-1].picId = _j.picUrl.split('http').length>1?_j.picUrl:_j.fileId;
                        that.positionData[i].picIds[parseInt(_j.visualType)-1].base64 = _j.picUrl.split('http').length>1?_j.picUrl:that.imgBaseUrl +_j.picUrl;
                      }
                      console.log('positionData',that.positionData);
                    }
                  }
                }
              }
              var len= that.positionData.filter(function(data){
                return data.isSelect;
              });
              if(len.length>0){
                that.firstPosition = len[0].repairCode;
              }
            }else if(data.code=='MATCH_DATA_FAIL'){
              that.$dialog.notify({mes: '车牌号没有开通玻璃保障服务', icon: 'error', timeout: 1200});
            }else{
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      setSubmitDate() {
        var _year = this.selectYear;
        var _month = parseInt(this.selectMonth)>9? this.selectMonth : '0' + parseInt(this.selectMonth);
        var _day = parseInt(this.selectDay)>9? this.selectDay : '0' + parseInt(this.selectDay) ;
        var _hourStar = parseInt(this.selectHour)>9?this.selectHour:'0'+parseInt(this.selectHour);
        var _hourEnd = parseInt(this.selectHour)+1>9?parseInt(this.selectHour)+1:'0'+(parseInt(this.selectHour)+1);
        var newTime = _year+'-'+ _month + '-' + _day + ' ';
        this.optApply.appointedStartTime = newTime + _hourStar+':00:00';
        this.optApply.appointedEndTime = newTime + _hourEnd+':00:00';
      },
      //保存预约信息
      submit(){
        
        let that = this;
        //提交数据
        if(that.optApply.customerName == ''){
          that.$dialog.notify({mes: '请输入姓名', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.phone == ''){
          that.$dialog.notify({mes: '请输入手机号码', icon: 'error', timeout: 1200});
          return;
        }else if(this.phoneStatus == "error"){
          that.$dialog.notify({mes: '请输入正确的手机号', icon: 'error', timeout: 1200});
          return;
        }
        var repairPosition = that.positionData.filter(function(data){
            return data.isSelect == true;
        })
        
        if(repairPosition.length==0){
          that.$dialog.notify({mes: '请选择维修部位', icon: 'error', timeout: 1200});
          return;
        }
         
        for(var i=0;i<repairPosition.length;i++){
            that.optApply.picArr[i] = {};
            that.optApply.picArr[i].repairCode = repairPosition[i].repairCode;
            that.optApply.picArr[i].picIds = [];
            for(var j=0;j<3;j++){
              var item = repairPosition[i].picIds[j];
              if(item.picId != ''){
                 var _opt = {};
                _opt.visualType = item.visualType;
                _opt.picId = item.picId;
                that.optApply.picArr[i].picIds.push(_opt);
              }
            }
        }
         
        this.$dialog.loading.open('保存中...');
        this.setSubmitDate();
        this.optApply.id=this.$route.query.id;
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_GLAGGSUPPORTEDIT_SAVE});
        this.$http.post(this.apiSubmitUrl,this.optApply)
          .then((response) => {
          const data = response.data;
          that.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
              removeSessionStore('glassSupportInfo');
              removeSessionStore('glassSupportPostion');
              removeSessionStore('serviceStore_btn_info');
              that.goTo();
            }});
            that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_GLAGGSUPPORTEDIT_SAVE_OK});
          }else{
            that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_GLAGGSUPPORTEDIT_SAVE_FAIL});
            if(data.code=='MATCH_DATA_FAIL'){
              that.$dialog.notify({mes: '无法匹配有效的开单记录', icon: 'error', timeout: 1200});
            }else if(data.code == "RECORD_DEALING"){
              that.$dialog.notify({mes: '预约信息已在处理中不允许修改', icon: 'error', timeout: 1200});
            }else if(data.code == "PARAM_ERROR"){
              that.$dialog.notify({mes: '找不到预约申请记录', icon: 'error', timeout: 1200});
            }else if(data.code == "POSITION_PICTURE_ISNULL"){
              that.$dialog.notify({mes: '维修部位和图片都为空', icon: 'error', timeout: 1200});
            }else if(data.code == "POSITION_PICTURE_INCOMPLETE"){
              that.$dialog.notify({mes: '请上传完整维修部位的图片', icon: 'error', timeout: 1200});
            }else if(data.code == "VISUAL_PICTURE_INCOMPLETE"){
              that.$dialog.notify({mes: '视觉类型图片显示不完整', icon: 'error', timeout: 1200});
            }else {
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          }
          
        },(response) => {
          setTimeout(()=>{
            that.$dialog.loading.close();
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          },300);
        });
      },
      //取消预约
      submitCancelApply(){
        var that = this;
        that.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_GLAGGSUPPORTEDIT_CANCEL});
        that.$http.post(this.apiCancelReserv,that.optCancelReserv)
            .then((response) => {
              const data = response.data;
              if(data.code == "S_OK") {
                that.$dialog.notify({mes: '取消预约成功', icon: 'error', timeout: 1200,callback: () => {
                  that.goTo();
                }});
                that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_GLAGGSUPPORTEDIT_CANCEL_OK});
              }else {
                that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_GLAGGSUPPORTEDIT_CANCEL_FAIL});
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }
            },(response) => {
              setTimeout(()=>{
                that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
              },300);
            });
      },
      submitData(pBol) {
        this.nowSelectData = this.selectYear+'年'+this.selectMonth+'月'+this.selectDay+'  ' + this.selectHour+':00'+'-'+ (parseInt(this.selectHour)+1) + ':00';
        if(pBol){
          this.tips2 = false;
        }
        
      },
      goTo(){
        this.$router.push({
          path: '/serviceInfo',
          query: {
            code:this.$route.query.code
          }
        });
      },
      //图片放大
      imgBig(str){
        document.getElementById(str).click();
      },
      //删除图片
      imgDelete(n,opt){
        this.positionData[n].picIds[parseInt(opt.visualType)-1].picId='';
        this.positionData[n].picIds[parseInt(opt.visualType)-1].base64='';
      },
      //获取本地图片信息
      readFile(n,str){ 
        var that = this;
        var file = document.getElementById(str).files[0]; //获取file对象
        //判断file的类型是不是图片类型。
        if(!/image\/\w+/.test(file.type)){ 
            alert("文件必须为图片！"); 
            return false; 
        }
        
        if(file){
          this.tips4 = true;
          this.$dialog.loading.open('上传中...');
          qiniuLib.uploadWithSDK(file,this.qiniuToken,(key,domain)=>{
            let imgUrl = domain+key;
            var _n = parseInt(str.split('_')[1]);
            that.positionData[n].picIds[_n-1].picId = imgUrl;
            that.positionData[n].picIds[_n-1].base64 = imgUrl;
            that.tips4 = false;
            that.$dialog.loading.close();
          });
        }
      },
       
      // //设置年份
      setYear(){
        var optDate = ydFunLib.getCurDate();
        if(!this.optApply.appointedStartTime){//没有选中的时间，默认为当前时间
           console.log('this.optApply.appointedStartTime1',this.optApply.appointedStartTime);
            var _opt = ydFunLib.getMextDay(optDate.nowYear,optDate.nowMonth,optDate.nowDay);
            this.selectYear = _opt.nowYear;
            this.selectDay = _opt.nowDay;
            this.selectMonth = _opt.nowMonth >9? _opt.nowMonth : '0' + _opt.nowMonth;
            this.curMonth = optDate.nowMonth;
        }else{//有选中的时间
          console.log('this.optApply.appointedStartTime2',this.optApply.appointedStartTime);
          var _date = this.optApply.appointedStartTime.split(' ')[0].split('-');
          this.selectYear = _date[0];
          this.selectDay = _date[2];
          this.selectMonth = _date[1];
          this.curMonth =  _date[1];
        }

        for(var i=0;i<10;i++){
          var n = i==0 ? optDate.nowYear: optDate.nowYear+i;
          this.yearList.push(n);
        }
      },
      //设置天数
      setDay(n){
        var optDate = ydFunLib.getCurDate();
        if(n){//显示当前时间开始的12个月日期
          for(var i=0;i<n;i++){
            if(i==0){//拿当前月份的数据
              var _day = ydFunLib.getThisMonthDay(optDate.nowYear,optDate.nowMonth);
              for(var j=1;j<_day+1;j++){
                var item = {};
                if(parseInt(optDate.nowDay)==j && optDate.nowHour<17){//当天
                  item.dayValue = j;
                  item.statu = 1;
                  item.yearValue = optDate.nowYear;
                  item.monthValue = optDate.nowMonth;
                  this.dayList.push(item);
                }else if(optDate.nowDay <j){
                  if((optDate.nowDay+1) == j){
                    item.statu = 1;
                    // console.log('selectMonth->',this.selectMonth,optDate.nowMonth);
                  }else{
                    item.statu = 3;
                  }
                  item.dayValue = j;
                  item.yearValue = optDate.nowYear;
                  item.monthValue = optDate.nowMonth;
                  this.dayList.push(item);
                }
              }
            }else{
              var _month = optDate.nowMonth+i;
              var _year = optDate.nowYear;
              if(_month>12){//当前月份是12月
                var _year = optDate.nowYear+1;
                var _day = ydFunLib.getThisMonthDay(optDate.nowYear+1,_month-12);
              }else{
                var _day = ydFunLib.getThisMonthDay(optDate.nowYear,optDate.nowMonth+i);
              }
              for(var j=1;j<_day;j++){
                var item = {};
                if(_month>12){
                  item.monthValue = _month-12;
                }else if(_month==12){
                  item.monthValue = 12;
                }else{
                  item.monthValue = _month;
                }
                item.dayValue = j;
                item.statu = 3;
                item.yearValue = _year;
                this.dayList.push(item);
              }
            }
          }
        }else{//显示当年
          for(var i=1;i<13;i++){
            var _day = ydFunLib.getThisMonthDay(this.selectYear,i);
            for(var j=1;j<_day+1;j++){
              var item = {};
              item.dayValue = j;
              item.monthValue = i;
              if(parseInt(this.selectYear) == nowYear) {//选择的是当前年份
                // console.log(1);
                if(parseInt(this.selectMonth) == i) {//当前年份当前月份
                  if(parseInt(this.selectDay) == j) {//当天
                    item.statu = 1;
                  }else if(parseInt(this.selectDay) < nowDay) {//已过,不可点
                    item.statu = 2;
                  }else{
                    item.statu = 3;
                  }
                }else if(i < parseInt(this.selectMonth)){
                  item.statu = 2;
                }else{
                  item.statu = 3;
                }
              }else{
                // console.log(2);
                if((parseInt(this.selectMonth) == i) && (parseInt(this.selectDay) == j)) {//当天
                  item.statu = 1;
                }else{//可点
                  item.statu = 3;
                }
              }
              this.dayList.push(item);
            }
          }
        }
      },
      //设置可选时间段
      setTime(){
        var optDate = ydFunLib.getCurDate();
        if(this.selectDay == optDate.nowDay){
          this.nowHour = optDate.nowHour;
          this.nowMinutes = optDate.nowMinutes;
          if(this.selectHour!=''){
            if(parseInt(this.selectHour)+1<optDate.nowHour){
              this.selectHour = optDate.nowHour;
            }
          }else{
            this.selectHour = optDate.nowHour;
          }
        }else if(this.optApply.appointedStartTime){
          var _date = this.optApply.appointedStartTime.split(' ')[1].split(':');
          this.selectHour = parseInt(_date[0]);
        }
        this.submitData(false);
      },
      //年份点击 
      yearBtn(n){
        this.selectYear = n;
      },
      //天数点击
      dayBtn(item,n){ 
        if(n==0){
          return false;
        }  
        var optDate = ydFunLib.getCurDate();
        this.selectYear = item.yearValue;
        this.selectMonth = item.monthValue;
        this.selectDay = item.dayValue;
        this.curMonth =  item.monthValue;
        if(this.selectYear==optDate.nowYear&&this.selectMonth==optDate.nowMonth&&this.selectDay==optDate.nowDay){
          this.isNowDay = true;
        }else{
          this.isNowDay = false;
        }
        this.setTime(item);
      },
      //时间段点击
      hourBtn(n) {
        this.selectHour = n;
      },
      //跳到指定位置
      goToScrollLeft(){
        //拿到当前屏幕下一个日期标签的宽度
        var spanWidth = document.getElementsByClassName('d_item')[0].clientWidth;
        //拿到当前屏幕两个日期标签距离屏幕左 边的距离差
        var spanOffetWidth = document.getElementsByClassName('d_item')[1].offsetLeft -document.getElementsByClassName('d_item')[0].offsetLeft;
        var itemDay = document.getElementById(Number.parseInt(this.selectMonth)+'_'+Number.parseInt(this.selectDay)).offsetLeft;
        document.getElementById("day").scrollLeft = itemDay-spanWidth-spanOffetWidth;
      },
      winScroll() {
        //横向滚动条距离屏幕左边的距离
        var t = document.getElementById("day").scrollLeft;
        var itemSpan = document.getElementsByClassName('d_item');
        //拿到当前屏幕下一个日期标签的宽度
        var spanWidth = document.getElementsByClassName('d_item')[0].clientWidth;
        //拿到当前屏幕两个日期标签距离屏幕左边的距离差
        var spanOffetWidth = document.getElementsByClassName('d_item')[1].offsetLeft -document.getElementsByClassName('d_item')[0].offsetLeft;
        // var item = parseInt(t/36)>0? parseInt(t/36)-1:0; //31为一个日期标签占据的大小
        if(t==0){
          var _month = itemSpan[0].attributes["id"].nodeValue.split('_')[0];
          this.curMonth = _month;
          // document.getElementById("month").innerText= _month;
        }else{
          // var item = (t+31)%36==0?parseInt((t+31)/36):parseInt((t+31)/36)+1; //31为一个日期标签占据的大小
          var item = parseInt((t+spanWidth)/spanOffetWidth); //31为一个日期标签占据的大小
          var _month = itemSpan[item].attributes["id"].nodeValue.split('_')[0];
          this.curMonth = _month;
          // document.getElementById("month").innerText= _month;
        }
      }
    }
  }
</script>