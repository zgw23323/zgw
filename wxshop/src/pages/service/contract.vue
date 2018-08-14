<template>
	<section class="service">
    <article class="s_contract">
      <p class="c_p">合同审查</p>
        <div class="i_top">
          <div class="item s_con" @click="openTip(2)">
            <div><span class="i_t">法律领域：</span><input type="text" v-model="optApply.legalfield" disabled readonly="readonly" placeholder="请选择专业领域" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="openTip(3)">
            <div><span class="i_t">当事方信息：</span><input type="text" v-model="optApply.purposereview" disabled readonly="readonly" placeholder="请选择主体" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="openTip(4)">
            <div><span class="i_t">审查目的：</span><input type="text" v-model="optApply.srivingaddress" disabled readonly="readonly" placeholder="0项" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con">
            <div><span class="i_t">联系人：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.contacts" placeholder="请输入联系人" />
            </div>
          </div>
          <div class="item no_bor s_con">
            <div><span class="i_t">手机号：</span><input type="number" @focus="handleFocus" v-on:blur="changeInput" maxlength="11" v-model="optApply.mobilephone" placeholder="请输入手机号" />
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor s_con">
            <!-- <div v-if="datetime2">期望文书交付时间：<input type="text"  @focus="tiemFocus" placeholder="请选择时间" /></div> -->
            <div v-show="datetime2">期望文书交付时间：<yd-datetime type="date" ref="datetime" v-model="optApply.documentStartTime" slot="right"></yd-datetime>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor s_con s_textarea">
            <div>咨询事项概述：</br><textarea @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.avcOverview" placeholder="详细描述事件情况，并告知你所期望额结果"></textarea>
            </div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor s_con">
            <div>上传附件：</br>
            <div class="file">
              <div class="img" v-for="item in imgList">
                <img @click="$refs.child.openTip(item.fileId)" v-bind:src="imgBaseUrl+'/shop/downLoad/showLogo/'+item.fileId+'?type=thumb'"/><i class="del_img" @click="deleteImg(item.fileId)">&nbsp;</i>
              </div>
              <div class="upImg" v-if="isFileBtn">
                <span class="l_add">&nbsp;</span> 
                <input type="file" id="file_input" @change="readFile" accept="image/*"/>
                <div class="p_20">添加附件</div>
              </div>
            </div>
            <p class="p_24">注意：上传更多的照片、扫描件便于精确审核</p>
            </div>
          </div>
        </div>
    </article>
    <div :class="{'d_call_phone dis':isKeyUp,'d_call_phone':!isKeyUp}" @click = "submit" v-if="btnShow">
      <a href="javascript: void(0);">提交审核</a>
    </div>
    <!--选择法律领域-->
    <yd-popup v-model="tips2" v-if="tips2" position="bottom" class="ser-legalfield" :height="height">
          <section class="absol">
            <div>
              <span @click="tipClose(2)">取消</span>
            </div>
            <div >
              <span @click="tipClose(2,'legalfield')">确定</span>
            </div>
          </section>  
          <ul class="shop">
            <li v-for="item in optReview.legalfield" @click="selectItem('legalfield',item)"><span>{{item}}</span><i :class="{'currGou':legalfield==item,'gou':legalfield!=item}">&nbsp;</i></li>
          </ul>
      </yd-popup>
      <!--选择当事方信息-->
      <yd-popup v-model="tips3" v-if="tips3" position="bottom" class="ser-legalfield" height="35%">
        <section class="absol">
          <div>
            <span @click="tipClose(3)">取消</span>
          </div>
          <div>
            <span @click="tipClose(3,'purposereview')">确定</span>
          </div>
        </section>  
        <ul class="shop">
          <li v-for="item in optReview.purposereview" @click="selectItem('purposereview',item)"><span>{{item}}</span><i :class="{'currGou':purposereview==item,'gou':purposereview!=item}">&nbsp;</i></li>
        </ul>
      </yd-popup>
      <!--选择审查目的--> 
      <yd-popup v-model="tips4" v-if="tips4" position="bottom" class="ser-legalfield" height="60%">
        <section class="absol">
          <div>
            <span @click="tipClose(4)">取消</span>
          </div>
          <div>
            <span @click="tipClose(4,'srivingaddress')">确定</span>
          </div>
        </section>  
        <ul class="shop">
          <li v-for="item in optReview.srivingaddress" @click="selectItem('srivingaddress',item)"><span>{{item}}</span><i :class="{'currGou':srivingaddress==item,'gou':srivingaddress!=item}">&nbsp;</i></li>
        </ul>
      </yd-popup>

      <yd-imgLoading-tip v-if="tips5"></yd-imgLoading-tip>
      <yd-imgBig-tip ref="child"></yd-imgBig-tip>
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        tips1: false,
        tips3:false,
        tips2:false,
        tips4:false,
        tips5:false,
        btnShow:true,
        title: '家庭律师',
        imgList:[],
        isFileBtn:true,
        isKeyUp:false,
        imgBaseUrl: window.imgBaseUrl,
        apiSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/wxrights/WxRightsContractReview?shopsid='+window.shopsid,//合同审核
        apiSelectDataUrl:window.baseUrl+window.appName+'/shopapi/func/wxrights/WxRightsContractReviewReturnList?shopsid='+window.shopsid,//获取下来选择数据类型
        apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=product&isBase64=y&flag=thumb',
        optReview:{
          legalfield:['交通人伤','房产购买','婚姻继承','劳动人事','债权债务','其他'],
          purposereview:['企业','个人'],
          srivingaddress:['风险规避','风险提示','表述清楚','其他']
        },
        legalfield:'',//选择的法律领域
        purposereview:'',//选择的当事方信息
        srivingaddress:'',//选择的审查目的
        phoneStatus:'',
        uNameStatus:'',
        datetime2: true,
        height:"75%",
        winHeight:0,
        documentStartTime:'',
        optApply:{
          legalfield:'',//法律领域
          purposereview:'',//当事方信息
          srivingaddress:'',//审查目的
          contacts:'',//联系人姓名
          mobilephone:'',//电话号码
          documentStartTime:'',// 日期时间
          avcOverview:'',//咨询事项概述
          filespaths:''//上传附件图片存地址多张图用逗号隔开
        }
      }
    },
    watch: {
      '$route.path': 'markInit',//监听当前路由变化（辅助初始化）
      'optApply.mobilephone'() { //监听当前输入手机号并实时校验
        if(ydFunLib.telInvalid(this.optApply.mobilephone)) {
          this.phoneStatus = "success";
          this.optApply.mobilephone = this.optApply.mobilephone.substring(0,11);
        }else {
          this.phoneStatus = "error";
        }
      },
       
      'optApply.contacts'() {
        if(!ydFunLib.uNameValid(this.optApply.contacts,true)) {  //监听当前输入用户名并实时校验
          this.uNameStatus = "success";
        }else {
          this.uNameStatus = "error";
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      // this.fetSelectData();
      this.getCurDate();
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
        var contractData = getSessionStore('contractSubmitInfo',true);
        if(typeof(contractData) !='boolean') {
          this.optApply = contractData;
          // this.datetime2 = false;
        }
        var imgData = getSessionStore('contractImgInfo',true);
        if(typeof(imgData) !='boolean'){
          this.imgList = imgData;
        }

        //判断设备，给定弹框高度
        var width_body = document.documentElement.clientWidth;
        var height_body = document.documentElement.clientHeight;
        if(width_body == 320 && height_body == 480){
            this.height = '82%';
        }

        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_CONTRACT});
      },
      handleFocus(){
        this.isKeyUp = true; 
        this.winHeight = document.documentElement.clientHeight;
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
        console.log('2222');
        this.isKeyUp = false;
        var that = this;
        // setTimeout(function(){
          // that.datetime2 = false;
          that.$refs.datetime.open();
        // },100);
        
      },
      tipClose(tips,str){
        this.isKeyUp = false;
        this['tips'+tips]=false;
        if(str){
          this.optApply[str] = this[str];
        }
      },
      //获取当前时间
      getCurDate(){
        var curDate = new Date();
        var y = curDate.getFullYear();
        var m = curDate.getMonth()+1>9?curDate.getMonth()+1:'0'+ (curDate.getMonth()+1);
        var d = curDate.getUTCDate()>9?curDate.getUTCDate():'0'+ curDate.getUTCDate();
        this.optApply.documentStartTime = y+'-'+m+'-'+d;
      },
      openTip(tips){
        this.isKeyUp = false;
        this['tips'+tips] = true;
      },
      //选择类型项
      selectItem(pStr,pItem){
        this[pStr] = pItem;
      },
      //删除图片
      deleteImg(fileId){
        console.log(fileId);
        var len = this.imgList.filter(function(data){
          return
        });
        for(var i=0;i<this.imgList.length;i++){
          if(fileId == this.imgList[i].fileId){
            this.imgList.splice(i,1);
            break;
          }
        }
        this.isFileBtn = true;
      },
      submit(){
        
        let that = this;
        //提交数据
        if(that.optApply.legalfield == ''){
          that.$dialog.notify({mes: '请选择法律领域', icon: 'error', timeout: 1200});
          return;
        }else{

        }
        if(that.optApply.purposereview == ''){
          that.$dialog.notify({mes: '请选择当事方信息', icon: 'error', timeout: 1200});
          return;
        }else{

        }
        if(that.optApply.srivingaddress == ''){
          that.$dialog.notify({mes: '请选择审查目的', icon: 'error', timeout: 1200});
          return;
        }else{

        }
        if(that.optApply.contacts == ''){
          that.$dialog.notify({mes: '联系人姓名', icon: 'error', timeout: 1200});
          return;
        }else if(this.uNameStatus == "error"){
          that.$dialog.notify({mes: '请输入正确的联系人姓名', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.mobilephone == ''){
          that.$dialog.notify({mes: '请输入电话号码', icon: 'error', timeout: 1200});
          return;
        }else if(this.phoneStatus == "error"){
          that.$dialog.notify({mes: '请输入正确的手机号', icon: 'error', timeout: 1200});
          return;
        }

        if(that.optApply.documentStartTime == ''){
          that.$dialog.notify({mes: '请输入期望交付时间', icon: 'error', timeout: 1200});
          return;
        }else{

        }
        if(that.optApply.avcOverview == ''){
          that.$dialog.notify({mes: '请输入咨询事项概述', icon: 'error', timeout: 1200});
          return;
        }else{

        }
        if(that.imgList.length==0){
          that.$dialog.notify({mes: '请上传证件照片', icon: 'error', timeout: 1200});
        }else{
          var len = [];
          that.imgList.filter(function(data){
            len.push(data.fileId);
          })
          that.optApply.filespaths = len.join(',');
        }
        console.log(this.optApply);
        this.$dialog.loading.open('保存中...');

        setSessionStore('contractImgInfo',that.imgList);//缓存上传的图片信息
        setSessionStore('contractSubmitInfo',this.optApply);//缓存合同审核确认信息
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_CONTRACT_SAVE});
        this.$http.post(this.apiSubmitUrl,JSON.stringify(this.optApply))
          .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
               that.goTo();
            }});
            that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_CONTRACT_APPLY_OK});
          }else {
            if(data.code == "COMMON_ADD_FAIL"){
              that.$dialog.notify({mes: '该手机号已经提交过审核信息', icon: 'error', timeout: 2000});
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200,callback: () => {
                 that.goTo();
              }});
            }
          }
        },(response) => {
          setTimeout(()=>{
            this.$dialog.loading.close();
            this.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
          },300);
        });
      },
      goTo(){
        // this.$router.push('/familyLawyer');
        this.$router.push({
          path: '/serviceInfo',
          query: {
            code:this.$route.query.code
          }
        });
      },
       //获取本地图片信息
      readFile(){ 
        var that = this;
        var file = document.getElementById('file_input').files[0]; //获取file对象
        //判断file的类型是不是图片类型。
        if(!/image\/\w+/.test(file.type)){ 
            alert("文件必须为图片！"); 
            return false; 
        }
        var reader = new FileReader(); //声明一个FileReader实例
        reader.readAsDataURL(file);
        reader.onload = function(e){ 
          ydFunLib.appendFile(this.result,function(base64){
              that.prodPicUpload(base64);
          });
        } 
      },
      //上传图片
      prodPicUpload(base64){
        var that = this;
        this.$dialog.loading.open('上传中...');
        this.tips5 = true;
        that.$http.post(this.apiUploadFileUrl, JSON.stringify({"imgStr":base64}))
            .then((response) => {
              const data = response.data;
              if(data.code == "S_OK") {
                var fileId = data["var"];
                    if(fileId){
                      var opt = {fileId:fileId};
                      that.imgList.push(opt);
                      if(that.imgList.length>=20){
                        that.isFileBtn = false;
                      }
                    }
              }else {
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }
              setTimeout(()=>{
                that.$dialog.loading.close();
                that.tips5 = false;
              },300);
            },(response) => {
              that.$dialog.loading.close();
              that.tips5 = false;
              setTimeout(()=>{
                that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
              },300);
            });
      },
      fetSelectData: function() {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiSelectDataUrl)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.optReview = data.var;
            }else {
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>