<template>
	<section class="service">
    <article class="s_contract dirving_sub">
      <p class="c_p">提交信息</p>
        <div class="i_top">
          <div class="item">
             <span class="i_t">驾照号：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" maxlength="18" v-model="optApply.sriving"  placeholder="请输入驾照号"/>
          </div>
          <div class="item">
            <span class="i_t">姓(拼音)：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.surname" maxlength="12" placeholder="例如（ZHANG）"/>
          </div>
          <div class="item">
            <span class="i_t">名(拼音)：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.name" placeholder="例如（XIAOMING）"/>
          </div>
          <div class="item" v-if="datetime1">
            <span class="i_t">出生日期：</span><input type="text"  @focus="tiemFocus(1)" placeholder="请输入出生日期" />
          </div>
          <div class="item" v-show="!datetime1">
            <span class="i_t">出生日期：</span><yd-datetime type="date" startDate="1917-01-01" ref="datetime1" v-model="optApply.birthdate" slot="right"></yd-datetime>
          </div>
          <div class="item">
            <span class="i_t">驾照上地址：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.srivingaddress" maxlength="100" placeholder="请输入驾照上地址" />
          </div>
          <div class="item">
            <span class="i_t">准驾车型：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.models" maxlength="50" placeholder="请输入准驾车型" />
          </div>
          <div class="item" v-if="datetime2">
            初次领证时间：<input type="text"  @focus="tiemFocus(2)" placeholder="请输入初次领证时间" />
          </div>
          <div class="item" v-show="!datetime2">
            初次领证时间：<yd-datetime type="date" startDate="1917-01-01" ref="datetime2" v-model="optApply.starttime" slot="right"></yd-datetime>
          </div>
          <div class="item" v-if="datetime3">有效起始日期：<input type="text"  @focus="tiemFocus(3)" placeholder="请输入有效起始日期" /></div>
          <div class="item" v-show="!datetime3">有效起始日期：<yd-datetime type="date" startDate="1917-01-01" ref="datetime3" v-model="optApply.endtime" slot="right"></yd-datetime>
          </div>
          <div class="item no_bor">
            <span class="i_t">有效年限：</span><input type="text" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.effective" placeholder="6" />
          </div>
        </div>
        <div class="i_bottom">
          <div class="item">
            <span class="i_t">收件人：</span><input type="text" v-model="optApply.addressee" @focus="handleFocus" v-on:blur="changeInput" placeholder="请输入收件人" />
          </div>
          <div class="item">
            <span class="i_t">手机号：</span><input type="number" v-model="optApply.cellphone" @focus="handleFocus" v-on:blur="changeInput" maxlength="11" placeholder="请输入手机号" />
          </div>
          <div class="item no_bor">
            <span class="i_t">收件地址：</span><input type="text" v-model="optApply.address" @focus="handleFocus" v-on:blur="changeInput" placeholder="请输入详细收件地址" />
          </div>
        </div>
        <div class="i_bottom">
          <div class="no_bor driving_sub_img">
            <div>证件上传：<span class="p_24">请上传驾照正反面照片</span></br>
              <div class="file">
                <div class="img" v-for="item in imgList"><img @click="$refs.child.openTip(item.fileId)" v-bind:src="imgBaseUrl+'/shop/downLoad/showLogo/'+item.fileId+'?type=thumb'"/><i class="del_img" @click="deleteImg(item.fileId)">&nbsp;</i></div>
                <div class="upImg" id="upImg" v-if="isFileBtn">
                  <span class="l_add">&nbsp;</span> 
                  <input type="file" id="file_input" @change="readFile" accept="image/*"/>
                  <div class="p_20">添加附件</div>
                </div>
              </div>
            </div>
          </div>
        </div>
    </article>
    <div :class="{'d_call_phone dis':isKeyUp,'d_call_phone':!isKeyUp}" @click = "submit" v-if="btnShow">
      <a href="javascript: void(0);">提交</a>
    </div>

    <yd-imgLoading-tip v-if="tips1"></yd-imgLoading-tip>
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
        title: '国际驾照/认证件',
        imgBaseUrl: window.imgBaseUrl,
        apiSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/wxrights/WxRightsDriversLicense?shopsid='+window.shopsid,//国际驾照提交信息
        apiUploadFileUrl:window.baseUrl+window.appName+'/shopapi/func/file/uploadFile?module=drivingLicenseSubmit&isBase64=y&flag=thumb',
        imgList:[],
        isFileBtn:true,
        datetime1: true,
        datetime2: true,
        datetime3: true,
        isKeyUp:false,
        tips1:false,
        btnShow:true,
        winHeight:0,
        optApply: {
          sriving:'',//驾照号
          surname:'',//姓
          name:'', //名
          birthdate:'',//  出生日期
          srivingaddress:'',// 驾照地址
          models:'',//   准驾车型
          starttime:'',// 初次领证时间
          endtime:'', //有效起始时间
          effective:'',//有效年限
          addressee:'',//收件人
          cellphone:'',// 收件人手机号
          address:'',//收件地址                                  
          documentspath:''// 证件上传地址string类型',多张图片用逗号分开
        }
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
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
        var drivingData = getSessionStore('drivingSubmitInfo',true);
        if(typeof(drivingData) !='boolean') {
          this.optApply = drivingData;
          this.datetime1 = false;
          this.datetime2 = false;
          this.datetime3 = false;
        }

        var imgData = getSessionStore('drivingImgInfo',true);
        if(typeof(imgData) !='boolean'){
          this.imgList = imgData;
          if(this.imgList.length==2){
            this.isFileBtn = false;
          }
        }
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_DRIVINGLICENSESUBMIT_SAVE});
      },
      tiemFocus(inx){
        this['datetime'+inx] = false;
        this.$refs['datetime'+inx].open();
        
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
      //获取当前时间
      getCurDate(){
        var curDate = new Date();
        var y = curDate.getFullYear();
        var m = curDate.getMonth()+1>9?curDate.getMonth()+1:'0'+ (curDate.getMonth()+1);
        var d = curDate.getUTCDate()>9?curDate.getUTCDate():'0'+ curDate.getUTCDate();
        var str = y+'-'+m+'-'+d;
        this.optApply.birthdate = str;
        this.optApply.starttime = str;
        this.optApply.endtime = str;
      },
      submit(){
       
        let that = this;
        //提交数据
        if(ydFunLib.isEmptyValid(that.optApply.sriving)){
          that.$dialog.notify({mes: '请输入驾照号', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.surname)){
          that.$dialog.notify({mes: '请输入您的姓', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.name)){
          that.$dialog.notify({mes: '请输入您的名', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.birthdate)){
          that.$dialog.notify({mes: '请输入您的出生日期', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.srivingaddress)){
          that.$dialog.notify({mes: '请输入您的驾照地址', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.models)){
          that.$dialog.notify({mes: '请输入您的准驾车型', icon: 'error', timeout: 1200});
          return;
        }else{
          if(ydFunLib.strlenValid(that.optApply.models) > 50) {
            that.$dialog.notify({mes: '输入车型号过长', icon: 'error', timeout: 1200});
            return;
          }
        }
        if(ydFunLib.isEmptyValid(that.optApply.starttime)){
          that.$dialog.notify({mes: '请输入初次领证时间', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.endtime)){
          that.$dialog.notify({mes: '请输入有效起始时间', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.effective)){
          that.$dialog.notify({mes: '请输入有效年限', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.addressee)){
          that.$dialog.notify({mes: '请输入收件人', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.cellphone)){
          that.$dialog.notify({mes: '请输入手机号码', icon: 'error', timeout: 1200});
          return;
        }else{
          if(!ydFunLib.telInvalid(that.optApply.cellphone)) {
            that.$dialog.notify({mes: '手机号输入有误', icon: 'error', timeout: 1200});
            return;
          }
        }
        if(ydFunLib.isEmptyValid(that.optApply.address)){
          that.$dialog.notify({mes: '请输入收件地址', icon: 'error', timeout: 1200});
          return;
        }
        if(that.imgList.length==0){
          that.$dialog.notify({mes: '请上传证件照片', icon: 'error', timeout: 1200});
        }else{
          var len = [];
          that.imgList.filter(function(data){
            len.push(data.fileId);
          })
          that.optApply.documentspath = len.join(',');
        }
          console.log(this.optApply);
          this.$dialog.loading.open('保存中...');

        setSessionStore('drivingImgInfo',that.imgList);//缓存上传的图片信息
        setSessionStore('drivingSubmitInfo',this.optApply);//缓存国际驾照资料确认信息
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_DRIVINGLICENSESUBMIT_SAVE});
        this.$http.post(this.apiSubmitUrl,JSON.stringify(this.optApply))
            .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
                that.goTo();
              }});
              that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_DRIVINGLICENSESUBMIT_APPLY_OK});
            }else {
              if(data.code == "COMMON_ADD_FAIL"){
                that.$dialog.toast({mes: '提交信息在处理中', icon: 'success', timeout: 2000 ,callback: () => {
                  that.goTo();
                }});
              }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      goTo(){
        // this.$router.push('/drivingLicense');
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
        this.tips1 = true;
        that.$http.post(this.apiUploadFileUrl, JSON.stringify({"imgStr":base64}))
            .then((response) => {
              const data = response.data;
              if(data.code == "S_OK") {
                var fileId = data["var"];
                  if(fileId){
                    var opt = {fileId:fileId};
                    that.imgList.push(opt);
                    if(that.imgList.length>=2){
                      that.isFileBtn = false;
                    }
                  }
              }else {
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }
              setTimeout(()=>{
                that.$dialog.loading.close();
                that.tips1 = false;
              },300);
            },(response) => {
              that.$dialog.loading.close();
              that.tips1 = false;
              setTimeout(()=>{
                that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
              },300);
            });
      },
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
      }
    }
  }
</script>