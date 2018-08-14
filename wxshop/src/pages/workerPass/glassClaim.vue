<template>
  <section class="service glassClaim">
    <article class="s_contract glassSupport s_glassClaim" style="padding-bottom: .22rem;">
        <div class="g_top">
          <div class="g_item glass_claim_head">
            <p class="p_name_1">客户信息</p>
            <div class="p_phone_1">
              <div class="p_item1"><span>姓<em class="e_kong">&nbsp;</em>名：</span>{{optSeachInfo.customerName}}</div>
              <div><span>手机号：</span>{{optSeachInfo.phone}}</div>
            </div>
            <div class="p_phone_1">
              <div><span>车牌号/车架号：</span>{{optSeachInfo.frameNumber}}</div>
              <!-- <div><span>车<em class="e_kong">&nbsp;</em>牌：</span>{{optSeachInfo.licensePlate}}</div> -->
            </div>
             <div class="p_phone_1"><div><span>服务时间：</span>{{optSeachInfo.appointedStartTime}} 至 {{optSeachInfo.appointedEndTime}}</div></div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor s_con s_textarea" style="padding: .37rem 0 .46rem;">
            <div>维修部位：</br>
              <ul class="pepair_parts" style="margin-bottom: 0;">
                <li :class="{'curBtn':item.isSelect}" v-for="item in positionData" @click="positionBtn(item)"><span>{{item.name}}</span></li>
              </ul>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor s_con">
            <div>上传照片</br>
              <div class="file_mess">
                  <p>注意：1、拍摄要求，正中拍摄一张，二侧从45度角各拍摄2张；</p>
                  <p style="margin-left:.72rem;margin-top:.13rem;">2、照片中需要有维修车辆车牌号码，且能看到车身；</p>
              </div>
              <ul class="img_demo">
                <li v-for="item in positionData" v-if="item.isSelect && item.repairCode==firstPosition" :class="'img_'+item.repairCode"></li>
              </ul>
              <div v-for="(item,index) in positionData" v-if="item.isSelect" class="glassSupport_flex">
                <div :class="{'upImg':imgData.picId=='','upImg no_bor':imgData.picId !=''}" v-for="imgData in item.picIds">
                  <!-- <input type="file" v-if="isUpdata" :id="item.repairCode+'_'+imgData.visualNum" @change="readFile('position',index,item.repairCode+'_'+imgData.visualNum)" capture="camera" accept="image/*"/> -->
                  <div v-if="imgData.picId!=''" class="img_look">
                    <img :src="imgData.base64" />
                    <div class="img_btn" v-if="isUpdata">
                      <i class="img_big_btn" @click="$refs.child.openTip(imgData.base64)" v-if="!isWx">&nbsp;</i>
                      <i class="img_big_btn" @click="imgBtn(imgData.base64)" v-if="isWx">&nbsp;</i>
                      <i class="img_delete_btn" @click="imgDelete(index,imgData.visualNum,imgData)">&nbsp;</i>
                    </div>
                  </div>
                  <div v-else>
                    <!-- <span>{{item.name}}</span> -->
                    <!-- <div class="p_20">{{imgData.message}}</div> -->
                    <form method="post" enctype="multipart/form-data" id="form" class="form" action="upload">
                      <span>{{item.name}}</span>
                      <a class="p_20" :id="item.repairCode+'_'+imgData.visualNum">{{imgData.message}}</a>
                    </form>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="g_item" style="padding: .3rem;">
            <div>维修工单上传：<br> 
              <div class="file_mess">
                  <p>注意：1、上传工单要明晰可见；</p>
                  <p style="margin-left:.72rem;margin-top:.13rem;">2、维修工单上需要有车主签名；</p>
              </div>
              <div class="file gong">
                <div class="img" style="" v-for="item in imgList">
                    <img :id="item.fileId" v-bind:src="item.base64"/>
                    <div class="img_btn" v-if="isUpdata">
                      <i class="img_big_btn" @click="$refs.child.openTip(item.base64)" v-if="!isWx">&nbsp;</i>
                      <i class="img_big_btn" @click="imgBtn(item.base64)" v-if="isWx">&nbsp;</i>
                      <i class="img_delete_btn" @click="deleteImg(item.base64,'orderPicArr')">&nbsp;</i>
                    </div>
                </div>
                <div class="upImg_gong" v-if="isUpdata">
                  <span class="l_up_file" style="margin-top: .2rem;">&nbsp;</span> 
                  <!-- <input type="file" id="file_input_gong" @change="readFile('gong')" accept="image/*"> 
                  <div class="p_20">上传维修单</div> -->
                  <form method="post" enctype="multipart/form-data" class='form' action="upload">
                      <div class="p_20">上传维修单</div>
                      <a class="file-input" id="orderPicArr"></a>
                  </form>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="g_item" style="padding: .3rem;">
            <div>结算清单上传：<br> 
              <div class="file_mess">
                  <p>注意：清单上车主签名、更换玻璃部位需要明晰可见；</p>
              </div>
              <div class="file gong">
                <div class="img" style="" v-for="item in jieImgList">
                    <img :id="item.fileId" v-bind:src="item.base64"/>
                    <div class="img_btn" v-if="isUpdata">
                      <i class="img_big_btn" @click="$refs.child.openTip(item.base64)" v-if="!isWx">&nbsp;</i>
                      <i class="img_big_btn" @click="imgBtn(item.base64)" v-if="isWx">&nbsp;</i>
                      <i class="img_delete_btn" @click="deleteImg(item.base64,'settlementPicArr')">&nbsp;</i>
                    </div>
                </div>
                <div class="upImg_gong" v-if="isUpdata">
                  <span class="l_up_file" style="margin-top: .2rem;">&nbsp;</span> 
                  <!-- <input type="file" id="file_input_jie" @change="readFile('jie')" accept="image/*">  -->
                  <!-- <div class="p_20">上传结算单</div> -->
                  <form method="post" enctype="multipart/form-data" class='form' action="upload">
                      <div class="p_20">上传结算单</div>
                      <a class="file-input" id="settlementPicArr"></a>
                  </form>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="g_item" style="padding: .3rem;">
            <div>故障描述：</br>
              <div class="m_textarea" v-if="isUpdata">
                <textarea v-model="optApply.repairDesc" maxlength="500" placeholder="请输入详细的故障描述..."></textarea>
                <div>{{maxLen}}/500</div>
              </div>
              <div class="m_textarea" v-else>
                <textarea v-model="optApply.repairDesc" readonly="readonly" placeholder="请输入详细的故障描述..."></textarea>
              </div>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor" style="padding-left: 0;" @click="selectReferrer('referrer')">
            <div><span class="i_t">业务员：</span>
            </div>
            <div><input type="text" v-model="optApply.salesmanName" style="text-align: right;margin-right: .2rem;" readonly="readonly" placeholder="请选择本次服务业务员" /><i class="right" v-if="isUpdata">&nbsp;</i></div>
          </div>
        </div>
        <div class="save_btn" @click = "submit" v-if="isUpdata&& flowStatus==0">保存</div>
        <div class="save_btn" @click = "submit" v-else-if="isUpdata && flowStatus==-1">重新提交审核</div>
        <div class="save_btn statu_mess" v-else-if="flowStatus==1">等待审核中</div>
        <div class="save_btn statu_mess" v-else-if="flowStatus==2">审核已通过</div>
    </article>
     <!-- 联系车主 -->
    <div class="flex_call_tip">
      <a :href="'tel:'+optSeachInfo.phone">&nbsp;
      </a> 
    </div>
    <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
    <yd-imgBig-tip ref="child"></yd-imgBig-tip>
  </section>
</template>
 
<script type="text/babel">
  import {setShopsId,setStore,setSessionStore,getSessionStore,removeSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  import ajaxModel from '../../utils/ajaxModel';
  import {qiniuLib} from '../../utils/qiniuLib';
  export default {
    data() {
      return {
        isWx:true,
        tips1: false,
        tips2:false,
        tips4:false,
        title: '玻璃保修服务',
        imgList:[],
        jieImgList:[],
        isUpdata:true,
        firstPosition:'',
        imgBaseUrl: window.imgBaseUrl,
        apiSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/insurance/wxRepairRecordManage?isSubmit=1',//保存
        // apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=glass&isBase64=y&flag=thumb',
        apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=insurance&recordSource=2& serviceType=BLBZSP&isBase64=y&flag=thumb',
        apiSeachInfoUrl:window.baseUrl+window.appName+'/shopapi/func/insurance/wxQueryApplyInfo?openId='+window.openId,//根据id查询索赔记录
        positionData:[
          {
            name:'前挡风玻璃',
            value:'1',
            isSelect:false,
            repairCode:'qiandangfengGlass',
            picIds:[
              {visualType:"1",visualNum:'1',picId:"",message:"拍摄左视角"},
              {visualType:"1",visualNum:'2',picId:"",message:"拍摄左视角"},
              {visualType:"2",visualNum:'3',picId:"",message:"拍摄中视角"},
              {visualType:"2",visualNum:'4',picId:"",message:"拍摄中视角"},
              {visualType:"3",visualNum:'5',picId:"",message:"拍摄右视角"},
              {visualType:"3",visualNum:'6',picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'后挡风玻璃',
            value:'2',
            isSelect:false,
            repairCode: 'houdangfengGlass',
            picIds:[
              {visualType:"1",visualNum:'1',picId:"",message:"拍摄左视角"},
              {visualType:"1",visualNum:'2',picId:"",message:"拍摄左视角"},
              {visualType:"2",visualNum:'3',picId:"",message:"拍摄中视角"},
              {visualType:"2",visualNum:'4',picId:"",message:"拍摄中视角"},
              {visualType:"3",visualNum:'5',picId:"",message:"拍摄右视角"},
              {visualType:"3",visualNum:'6',picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右前门玻璃',
            value:'3',
            isSelect:false,
            repairCode: 'zuoyouqianmenGlass',
            picIds:[
              {visualType:"1",visualNum:'1',picId:"",message:"拍摄左视角"},
              {visualType:"1",visualNum:'2',picId:"",message:"拍摄左视角"},
              {visualType:"2",visualNum:'3',picId:"",message:"拍摄中视角"},
              {visualType:"2",visualNum:'4',picId:"",message:"拍摄中视角"},
              {visualType:"3",visualNum:'5',picId:"",message:"拍摄右视角"},
              {visualType:"3",visualNum:'6',picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右中门玻璃',
            value:'4',
            isSelect:false,
            repairCode: 'zuoyouzhongmenGlass',
            picIds:[
              {visualType:"1",visualNum:'1',picId:"",message:"拍摄左视角"},
              {visualType:"1",visualNum:'2',picId:"",message:"拍摄左视角"},
              {visualType:"2",visualNum:'3',picId:"",message:"拍摄中视角"},
              {visualType:"2",visualNum:'4',picId:"",message:"拍摄中视角"},
              {visualType:"3",visualNum:'5',picId:"",message:"拍摄右视角"},
              {visualType:"3",visualNum:'6',picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右尾侧玻璃',
            value:'5',
            isSelect:false,
            repairCode: 'zuoyouweiceGlass',
            picIds:[
              {visualType:"1",visualNum:'1',picId:"",message:"拍摄左视角"},
              {visualType:"1",visualNum:'2',picId:"",message:"拍摄左视角"},
              {visualType:"2",visualNum:'3',picId:"",message:"拍摄中视角"},
              {visualType:"2",visualNum:'4',picId:"",message:"拍摄中视角"},
              {visualType:"3",visualNum:'5',picId:"",message:"拍摄右视角"},
              {visualType:"3",visualNum:'6',picId:"",message:"拍摄右视角"}
            ]
          },
          {
            name:'左右小三角玻璃',
            value:'6',
            isSelect:false,
            repairCode: 'zuoyouxiaosanjiaoGlass',
            picIds:[
              {visualType:"1",visualNum:'1',picId:"",message:"拍摄左视角"},
              {visualType:"1",visualNum:'2',picId:"",message:"拍摄左视角"},
              {visualType:"2",visualNum:'3',picId:"",message:"拍摄中视角"},
              {visualType:"2",visualNum:'4',picId:"",message:"拍摄中视角"},
              {visualType:"3",visualNum:'5',picId:"",message:"拍摄右视角"},
              {visualType:"3",visualNum:'6',picId:"",message:"拍摄右视角"}
            ]
          }
        ],
        positionImgData:[],
        optSeachInfo:{
          customerName:'',
          appointedStartTime:'',
          appointedEndTime:''
        },
        isHasData:false,
        flowStatus: 0,
        maxLen:0,
        qiniuToken:'',
        optApply:{
          applyId:'',//售后预约ID
          insuranceOpenId:'',//匹配的开通资料
          salesmanId:'',//业务员ID
          salesmanName:'',//业务员名称
          repairDesc:'',// 故障描述
          orderPicArr:'',//维修工单文件id
          settlementPicArr:'',//结算清单文件id
          picArr:[]//维修部位照片组合
        }
      }
    },
    watch: {
      '$route.path': 'markInit',//监听当前路由变化（辅助初始化）
      'optApply.repairDesc'() {
        var len = ydFunLib.strlenValid(this.optApply.repairDesc);
        if(len<=500) {  //监听当前输入用户名并实时校验
          this.maxLen = len;
        }else {
          this.optApply.repairDesc = ydFunLib.subStrFun(this.optApply.repairDesc,500);
          this.maxLen = ydFunLib.strlenValid(this.optApply.repairDesc);
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      let that = this;
      ajaxModel.generateToken(this,(result)=>{
        that.qiniuToken = result.qiniuToken;
        that.newUploadFileFun(['settlementPicArr','orderPicArr']);
        that.setImgNewProject();
      },'?tokenType=qiniu')
      this.fecthClaimInfo();
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
     
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        document.body.scrollTop = document.documentElement.scrollTop = 0;
        var s_1 = getSessionStore('s_1',false);
        if(typeof(s_1) !='boolean'){
          removeSessionStore('s_1');
        }
         //拿到用户选择的维修部位信息
        var positionData = getSessionStore('glassClaimPostion',true);
        if(typeof(positionData) !='boolean'){
          this.positionData = positionData;
          console.log(this.positionData);
        }

        //拿到缓存的维修工单图片信息
        var _imgList = getSessionStore('glassClaimImg',true);
        if(typeof(_imgList) !='boolean') {
          this.imgList = _imgList;
        }

        //拿到缓存的结算清单图片信息
        var _jieImgList = getSessionStore('settlementPicImg',true);
        if(typeof(_jieImgList) !='boolean') {
          this.jieImgList = _jieImgList;
        }
        
        //从缓存中拿到提交基本信息
        var glassClaimApply = getSessionStore('glassClaimApply',true);
        if(typeof(glassClaimApply) !='boolean') {
          console.log('glassReferrer',glassClaimApply);
          this.optApply = glassClaimApply;
        }
        //从缓存中拿到推荐人信息
        var glassReferrer = getSessionStore('glass_select_referrer',true);
        if(typeof(glassReferrer) !='boolean') {
          console.log('glassReferrer',glassReferrer);
          this.optApply.salesmanId = glassReferrer.id.toString();
          this.optApply.salesmanName = glassReferrer.realName;
        }
        //判断当前是否是微信浏览器
        this.isWx = ydFunLib.curNavigator();
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_GLASSCLAIM});
      },
      //根据ID查询索赔记录信息
      fecthClaimInfo(){
        var that = this;
        var _id = window.location.href.split('glassClaim?')[1].split('id=')[1].split('&')[0];
        this.$dialog.loading.open('加载中...');
        this.$http.post(this.apiSeachInfoUrl,{'id':_id})
          .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.optSeachInfo = data.var.applyInfo;
            that.optSeachInfo.appointedStartTime = data.var.openInfo.startDateString;
            that.optSeachInfo.appointedEndTime = data.var.openInfo.endDateString;
            that.optApply.insuranceOpenId = data.var.openInfo.openId;

            var _type = this.$route.query.type;
            if(data.var.repair !=null) {
              that.flowStatus = data.var.repair.flowStatus;
              that.isHasData = true;
              if(!_type){//如果是从推荐人回到的页面，数据以缓存中的为准
                that.optApply.repairDesc = data.var.repair.repairDesc;
                console.log('orderMap',data.var.repair.orderMap);
                for(var key in data.var.repair.orderMap){
                  var _img = that.imgList.filter(function(data){
                      return data.fileId == key;
                  });
                  if(_img.length == 0){
                    var opt = {};
                    opt.fileId = key;
                    let _orderMap = data.var.repair.orderMap[key];
                    opt.base64 = _orderMap.split('http').length>1? _orderMap:that.imgBaseUrl + _orderMap;
                    that.imgList.push(opt);
                  }
                }
                for(var key in data.var.repair.settlementMap){
                  var _img = that.jieImgList.filter(function(data){
                      return data.fileId == key;
                  });
                  if(_img.length == 0){
                    var opt = {};
                    opt.fileId = key;
                    let _settlementMap = data.var.repair.settlementMap[key];
                    opt.base64 = _settlementMap.split('http').length>1 ?_settlementMap : that.imgBaseUrl + _settlementMap;
                    that.jieImgList.push(opt);
                  }
                }
                that.optApply.salesmanId = data.var.repair.salesmanId;
                that.optApply.salesmanName = data.var.repair.salesmanName;
                that.setPositionBtn(data.var.repair.fileMap);
              }
              that.optApply.id = data.var.repair.id;
              that.isUpdata = data.var.repair.flowStatus != -1 ? false:true;
            }else if(!_type){
              that.setPositionBtn(data.var.applyInfo.fileMap);
            }
          }else{
            if(data.code =="PARAM_ERROR"){
              that.$dialog.notify({mes: '找不到预约申请记录', icon: 'error', timeout: 1200});
            }else if(data.code == "NO_OPEN_SERVICE"){
              that.$dialog.notify({mes: '未开通索赔权限', icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          setTimeout(()=>{
            this.$dialog.loading.close();
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          },300);
        });
      },
      newUploadFileFun(arr){
        let that = this;
        for(var i=0;i<arr.length;i++){
          // let item = arr[i];
          qiniuLib.uploadWithSDK1(that.qiniuToken,arr[i],this,(key,strId) =>{
            console.log('strId',window.qiniuDomain+key);
            that.$dialog.loading.close();
            var imgUrl = window.qiniuDomain+key;
            var opt = {fileId:imgUrl,base64:imgUrl};
            if(strId == 'settlementPicArr'){
              that.optApply[strId] = window.qiniuDomain+key;
              that.jieImgList.push(opt);
              // that.isAgreementFileShowBtn=false;
            }else if(strId == 'orderPicArr'){
              that.optApply[strId] = window.qiniuDomain+key;
              that.imgList.push(opt);
              // that.isCostFileShowBtn=false;
            }else{
              let str = strId.split('_');
              let _positionData = that.positionData.filter((data)=>{
                return data.repairCode == str[0];
              });
              console.log('strId',window.qiniuDomain+key);
              _positionData[0].picIds[parseInt(str[1])-1].picId = imgUrl;
              _positionData[0].picIds[parseInt(str[1])-1].base64 = imgUrl;
            }
          });
        }
      },
      //实例化上传图片
      setImgNewProject(){
        let that = this;
        console.log('setImgNewProject',this.positionData);
        for(var j=0;j<this.positionData.length;j++){
          console.log('setImgNewProject11');
          let item = this.positionData[j];
          console.log('tem.isSelect',item.isSelect);
          if(item.isSelect){
            console.log('tem.isSelect',item.isSelect);
            for(var i=0;i<item.picIds.length;i++){
              let arrItem = item.picIds[i];
              let str = item.repairCode + '_' + arrItem.visualNum;
              qiniuLib.uploadWithSDK1(that.qiniuToken,str,this,(key,strId) =>{
                console.log('strId',window.qiniuDomain+key);
                that.$dialog.loading.close();
                var imgUrl = window.qiniuDomain+key;
                  let str = strId.split('_');
                  let _positionData = that.positionData.filter((data)=>{
                    return data.repairCode == str[0];
                  });
                  console.log('strId',window.qiniuDomain+key);
                  _positionData[0].picIds[parseInt(str[1])-1].picId = imgUrl;
                  _positionData[0].picIds[parseInt(str[1])-1].base64 = imgUrl;
              });
            }
          }
        }
      },

      //维修部位选择
      positionBtn(opt){
        if(this.isUpdata) {
          this.positionData[parseInt(opt.value)-1].isSelect = !this.positionData[parseInt(opt.value)-1].isSelect;
          this.setImgDataList();
        }
      },
      imgBtn(pId){
        ydFunLib.wxBigImg(pId);
      },
      //设置维修部位与对应的照片
      setPositionBtn(opt) {
        for(var key in opt){
            for(var i=0;i<this.positionData.length;i++){
              var _i = this.positionData[i];
              if(key==_i.repairCode){
                this.positionData[i].isSelect = true;
                for(var j=0;j<opt[key].length;j++){
                  var _j = opt[key][j];
                  this.positionData[i].picIds[parseInt(_j.visualType)-1].picId = _j.fileId;
                  this.positionData[i].picIds[parseInt(_j.visualType)-1].base64 = _j.picUrl.split('http').length>1?_j.picUrl:this.imgBaseUrl +_j.picUrl;
                }
              }
            }
        }
        var len= this.positionData.filter(function(data){
          return data.isSelect;
        });
        if(len.length>0){
          this.firstPosition = len[0].repairCode;
        }
        this.setImgNewProject();
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
        console.log('len1',len);
        if(len.length>0){
          this.firstPosition = len[0].repairCode;
        }
      },
      glassOpenOrderInfo(){
        setSessionStore('glassClaimPostion',this.positionData);
        setSessionStore('glassClaimImg',this.imgList);
        setSessionStore('settlementPicImg',this.jieImgList);
        setSessionStore('glassClaimApply',this.optApply);
      },
       //跳去选择推荐人
      selectReferrer(str) {
        this.glassOpenOrderInfo();
        var _id = window.location.href.split('glassClaim?')[1].split('id=')[1].split('&')[0];
        // var _openId = window.location.href.split('glassClaim?')[1].split('openId=');
        var _openId = window.openId;
        var opt = {from: 'glassClaim',id: _id,orgId:this.optSeachInfo.orgId};
        setSessionStore('s_1',str);
        if(_openId.length>1){
          opt.openId = _openId[1].split('&')[0];
        }
        //跳去地址
        this.$router.push({
          path: '/'+str,
          query: opt
        });
      },
      
      //保存预约信息
      submit(){
        let that = this;
        //提交数据
        var repairPosition = that.positionData.filter(function(data){
            return data.isSelect == true;
        })
        if(repairPosition.length==0){
          that.$dialog.notify({mes: '请选择维修部位', icon: 'error', timeout: 1200});
          return;
        }
        //维修部位图片组合
        var isBack = false;
        for(var i=0;i<repairPosition.length;i++){
          that.optApply.picArr[i] = {};
          that.optApply.picArr[i].repairCode = repairPosition[i].repairCode;
          that.optApply.picArr[i].picIds = [];
          for(var j=0;j<repairPosition[i].picIds.length;j++){
            var item = repairPosition[i].picIds[j];
            if(item.picId != ''){
              var _opt = {};
              _opt.visualType = item.visualType;
              _opt.picId = item.picId;
              that.optApply.picArr[i].picIds.push(_opt);
            }else{
              isBack = true;
              break;
            }
          }
          if(isBack){
            break;
          }
        }
        if(isBack){
          that.$dialog.notify({mes: '请上传完整的维修部位照片', icon: 'error', timeout: 1200});
          return;
        }
        //上传工单图片组合
        if(that.imgList.length==0){
          that.$dialog.notify({mes: '请上传维修工单照片', icon: 'error', timeout: 1200});
          return;
        }else{
          var len = [];
          that.imgList.filter(function(data){
            len.push(data.fileId);
          })
          that.optApply.orderPicArr = len;
        }
        //上传结算图片组合
        if(that.jieImgList.length==0){
          that.$dialog.notify({mes: '请上传结算清单照片', icon: 'error', timeout: 1200});
          return;
        }else{
          var len = [];
          that.jieImgList.filter(function(data){
            len.push(data.fileId);
          })
          that.optApply.settlementPicArr = len;
        }
        if(that.optApply.salesmanId==''){
          that.$dialog.notify({mes: '请选择业务员', icon: 'error', timeout: 1200});
          return;
        }
        that.optApply.applyId = that.optSeachInfo.id;
        setSessionStore('test-glassClaim',that.optApply);
        // return false;
        this.$dialog.loading.open('保存中...');
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.BTN_PAGE_GLASSCLAIM_SAVE});
        var _url = that.isHasData ? this.apiSubmitUrl +'&type=update': this.apiSubmitUrl +'&type=add';
        var _wUrl = window.location.href.split('glassClaim?')[1].split('openId=');
        var _openId = window.openId;
        this.$http.post(_url+'&openId='+_openId,this.optApply)
          .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 2000 ,callback: () => {
               that.goTo(_openId);
            }});
            that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_GLASSCLAIM_SAVE_OK});
            //保存成功删除缓存信息
            removeSessionStore('glassClaimPostion');
            removeSessionStore('glassClaimImg');
            removeSessionStore('glassClaimApply');
            removeSessionStore('glass_select_referrer');
            removeSessionStore('settlementPicImg');
          }else {
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_GLASSCLAIM_SAVE_FAIL});
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
        },(response) => {
          setTimeout(()=>{
            this.$dialog.loading.close();
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          },300);
        });
      },
       
      goTo(openId){
        if(window.workerHomeUrl){
          location.href = window.workerHomeUrl;
        }else{
          setStore('_openId',openId);
          setStore('_shopsid',openId);
          this.$router.push({
            path: '/infoPushApply',
            query: {
               'openId': openId
            }
          });
        }
      },
      //图片放大
      imgBig(str){
        document.getElementById(str).click();
      },
      //删除图片
      imgDelete(n1,n2,opt){
        this.positionData[n1].picIds[parseInt(n2)-1].picId='';
        this.positionData[n1].picIds[parseInt(n2)-1].base64='';
        setTimeout(()=>{
          this.newUploadFileFun([this.positionData[n1].repairCode+'_'+n2]);
        },200);
      },

      //删除图片-维修工单
      deleteImg(fileId,mess){
        if(mess=='orderPicArr'){
          var len = this.imgList.filter(function(data){
            return data.base64 != fileId;
          });
          this.imgList = len;
        }else{
          var len = this.jieImgList.filter(function(data){
            return data.base64 != fileId;
          });
          this.jieImgList = len;
        }
        // this.newUploadFileFun([mess]);
      },
      //获取本地图片信息
      readFile(type,n,str){ 
        var that = this;
        var file = type =='position' ? document.getElementById(str).files[0]:document.getElementById('file_input_'+type).files[0]; //获取file对象
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
            if(type=='gong'){
              var opt = {
                fileId:imgUrl,
                base64:imgUrl
              };
              that.imgList.push(opt);
            }else if(type == 'jie'){
              var opt = {
                fileId:imgUrl,
                base64:imgUrl
              };
              that.jieImgList.push(opt);
            }else{
              var _n = parseInt(str.split('_')[1]);
              that.positionData[n].picIds[_n-1].picId = imgUrl;
              that.positionData[n].picIds[_n-1].base64 = imgUrl;
            }
            that.tips4 = false;
            that.$dialog.loading.close();
          });
        }
      }
    }
  }
</script>