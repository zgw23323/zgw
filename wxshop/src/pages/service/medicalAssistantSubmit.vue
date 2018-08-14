<template>
	<section class="service">
    <article class="s_contract maintain_sub" style="padding-bottom: 3rem;">
        <div class="i_top">
          <div class="item">
             <span>姓名：</span><input type="text" class="m_w" @focus="handleFocus" v-on:blur="changeInput" maxlength="50" v-model="optApply.cusname"  placeholder="请输入车主姓名"/>
          </div>
          <div class="item">
            <span>手机号：</span><input type="number" class="m_w" v-model="optApply.phone" @focus="handleFocus" v-on:blur="changeInput" maxlength="11" placeholder="请输入您服务合同上预留的手机号" />
          </div>
          <div class="item">
            <span>身份证：</span><input type="text" class="m_w" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.idcard" maxlength="50" placeholder="请输入您的身份证号码" />
          </div>
          <div class="item">
            <span>医保卡：</span><input type="text" class="m_w" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.cardno" maxlength="50" placeholder="请输入您的医保卡" />
          </div>
          <div class="item no_bor maintain_time" v-if="datetime2" @click="tiemFocus(2)">
            预约时间：<input type="text" class="m_w" placeholder="选择预约时间（请提前24小时）" />
            <i class="time_logo">&nbsp;</i>
          </div>
          <div class="item no_bor maintain_time" v-show="!datetime2">
            <span class="i_t">预约时间：</span><yd-datetime type="date" startDate="1917-01-01" ref="datetime2" v-model="optApply.befortime" slot="right"></yd-datetime><i class="time_logo">&nbsp;</i>
          </div>
        </div>
        <div class="maintain-type">
          <div class="item s_con" @click="openTip(1)">
            <div><span>服务类型：</span><input type="text" v-model="optsData.befortype" disabled readonly="readonly" placeholder="请选择服务类型" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="goToHospital">
            <div><span>选择医院：</span><input type="text" v-model="optsData.hospitalName" disabled readonly="readonly" placeholder="请选择服务医院" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item s_con" @click="openTip(2)">
            <div><span>诊断类型：</span><input type="text" v-model="optsData.illnesstype" disabled readonly="readonly" placeholder="请选择诊断类型" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
          <div class="item no_bor s_con" @click="openTip(3)">
            <div><span>相关病史：</span><input type="text" v-model="optsData.operation" disabled readonly="readonly" placeholder="请选择是否有相关病情的手术史" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="item no_bor" style="justify-content: flex-start;">
             <span>预约科室：</span><input type="text" class="m_w" @focus="handleFocus" v-on:blur="changeInput" maxlength="50" v-model="optsData.describe1"  placeholder="请输入要预约的科室"/>
          </div>
        </div>
        <div class="i_bottom">
          <div class="main_addres no_bor s_con">
            <div>病情描述：</br>
              <div class="m_textarea">
                <textarea @focus="handleFocus" v-on:blur="changeInput" v-model="optsData.describe2" placeholder="请描述病情"></textarea>
              </div>
            </div>
          </div>
        </div>
        <!-- <div class="s_user_notice" style="margin-top: .3rem;">
          <i class="notice_logo">&nbsp;</i>
          <span class="notice_mess">已阅读<span>使用须知</span></span>
        </div> -->
    </article>
    <!--服务类型-->
    <yd-popup v-model="tips1" position="bottom" class="ser-legalfield" height="40%">
      <section class="absol">
        <div>
          <span @click="tipClose(1)">取消</span>
        </div>
        <div>
          <span @click="tipClose(1,'befortype')">确定</span>
        </div>
      </section>  
      <ul class="shop">
        <li v-for="item in optSelect.befortype" @click="selectItem('befortype',item)"><span>{{item.name}}</span><i :class="{'currGou':optsData.befortype==item.name,'gou':optsData.befortype!=item.name}">&nbsp;</i></li>
      </ul>
    </yd-popup>
    <!--诊断类型-->
    <yd-popup v-model="tips2" position="bottom" class="ser-legalfield" height="40%">
      <section class="absol">
        <div>
          <span @click="tipClose(2)">取消</span>
        </div>
        <div>
          <span @click="tipClose(2,'illnesstype')">确定</span>
        </div>
      </section>  
      <ul class="shop">
        <li v-for="item in optSelect.illnesstype" @click="selectItem('illnesstype',item)"><span>{{item.name}}</span><i :class="{'currGou':optsData.illnesstype==item.name,'gou':optsData.illnesstype!=item.name}">&nbsp;</i></li>
      </ul>
    </yd-popup>
    <!--诊断类型-->
    <yd-popup v-model="tips3" position="bottom" class="ser-legalfield" height="40%">
      <section class="absol">
        <div>
          <span @click="tipClose(3)">取消</span>
        </div>
        <div>
          <span @click="tipClose(3,'operation')">确定</span>
        </div>
      </section>  
      <ul class="shop">
        <li v-for="item in optSelect.operation" @click="selectItem('operation',item)"><span>{{item.name}}</span><i :class="{'currGou':optsData.operation==item.name,'gou':optsData.operation!=item.name}">&nbsp;</i></li>
      </ul>
    </yd-popup>
    <div :class="{'flex_bottom dis':isKeyUp,'flex_bottom':!isKeyUp}" v-if="btnShow">
      <div class="s_notice">
        <i class="notice_logo">&nbsp;</i>
        <span class="notice_mess">已阅读<span>使用须知</span></span>
      </div>
      <div class="m_submit" @click = "submit">
        <a href="javascript: void(0);">提交</a>
      </div>
    </div>
  </section>
</template>
<script type="text/babel">
  import {MP} from './../../modules/map/index';
  import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        // title: '就医帮手-绿色通到',
        title: '就医帮手',
        apiSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/wxrights/WxRightsOverviewOrder?shopsid='+window.shopsid,//就医帮手预约
        start1:false,
        isFileBtn:true,
        datetime1: true,
        datetime2: true,
        datetime3: true,
        isKeyUp:false,
        tips1:false,
        tips2:false,
        tips3:false,
        btnShow:true,
        winHeight:0,
        optsData: {
          befortype:'',
          illnesstype:'',
          operation:'',
          hospitalName:'',//医院名称
          describe1:'',//预约科室
          describe2:''//病情描述
        },
        optSelect:{
          befortype:[{'value':1,name:'绿色通道'},{value:2,name:'尊享陪诊'}],//服务类型
          illnesstype:[{'value':1,name:'初诊'},{value:2,name:'复诊'}],//诊断类型
          operation:[{'value':1,name:'否'},{value:2,name:'是'}]//是否有相关手速史
        },
        beforeTime:'',
        befortype:'',
        illnesstype:'',
        operation:'',
        model2:'',
        hospitalName:'',
        describe1:'',//预约科室
        describe2:'',//病情描述
        optApply: {
          cusname:'',//客户姓名(必填)
          phone:'',//（激活时手机号，必填）
          idcard:'',//身份证号（必填）
          befortime:'',//预约时间（提前7个工作日）
          befortype:'',//服务类型：1绿色通道2陪诊（必填）       
          province:'1',//省份id（必填）
          city:'1',//市id（必填）
          hospital:'1',//医院id（必填）
          illnesstype:'',//诊断类型：1初诊2复诊（必填）
          operation:'',//是否有相关病情的手术：1否2是（必填）
          describe:'',//要预约的科室和病情描述（必填）
          cardno:''//医保卡（选填）
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
        var drivingData = getSessionStore('medicalSubmitInfo',true);
        if(typeof(drivingData) !='boolean') {
          this.optApply = drivingData.optApply;
          this.optsData = drivingData.optsData;
          this.datetime2 = false;
          this.beforeTime = this.optApply.beforeTime;
          
        }

        var hospitalData = getSessionStore('hospitalInfo',true);
        if(typeof(imgData) !='boolean'){
          this.optsData.hospitalName = hospitalData.name;
          this.optApply.province = hospitalData.provinceId;
          this.optApply.city = hospitalData.cityId;
          this.optApply.hospital = hospitalData.id;
        }
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_MEDICALASSISTANTSUBMIT});
      },
      openTip(tips){
        this['tips'+tips] = true;
      },
       //选择类型项
      selectItem(pStr,pItem){
        this.optsData[pStr] = pItem.name;
        this.optApply[pStr] = pItem.value;
      },
      tipClose(tips,str){
        this['tips'+tips]=false;
      },
      goToHospital(){
        //跳去选择医院缓存录入信息
        setSessionStore('medicalSubmitInfo',{'optApply':this.optApply,'optsData':this.optsData});
        this.$router.push({
            path: 'hospital',
            query: {
              type: '1'
            }
        });
      },  
      tiemFocus(inx){
        this.beforeTime = this.optApply.beforeTime;
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
        this.optApply.befortime = str;
      },
      submit(){
        
        let that = this;
        //提交数据
        if(ydFunLib.isEmptyValid(that.optApply.cusname)){
          that.$dialog.notify({mes: '请输入姓名', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.phone)){
          that.$dialog.notify({mes: '请输入您服务合同上的手机号', icon: 'error', timeout: 1200});
          return;
        }else{
          if(!ydFunLib.telInvalid(that.optApply.phone)) {
            that.$dialog.notify({mes: '输入手机号不正确', icon: 'error', timeout: 1200});
            return;
          }
        }
        if(ydFunLib.isEmptyValid(that.optApply.idcard)){
          that.$dialog.notify({mes: '请输入您的身份证号码', icon: 'error', timeout: 1200});
          return;
        }
        if(/(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/.test(that.optApply.idcardr) === false) {
          that.$dialog.notify({mes: '身份证输入有误', icon: 'error', timeout: 1200});
          return;
        }
        if(that.beforeTime == ''){
          that.$dialog.notify({mes: '请选择预约时间', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.befortype == ''){
          that.$dialog.notify({mes: '请选择服务类型', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.hospital == ''){
          that.$dialog.notify({mes: '请选择要预约的医院', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.illnesstype == ''){
          that.$dialog.notify({mes: '请选择诊断类型', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.operation == ''){
          that.$dialog.notify({mes: '请选择是否有相关病史', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optsData.describe1)){
          that.$dialog.notify({mes: '请输入预约科室', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optsData.describe2)){
          that.$dialog.notify({mes: '请输入病情相关描述', icon: 'error', timeout: 1200});
          return;
        }
         
        console.log(this.optApply);
        this.$dialog.loading.open('保存中...');
        this.optApply.describe = "预约科室："+this.describe1+';病情相关描述:'+this.describe2;
        setSessionStore('medicalSubmitInfo',{'optApply':this.optApply,'optsData':this.optsData});//缓存国际驾照资料确认信息
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_MEDICALASSISTANTSUBMIT_SAVE});
        this.$http.post(this.apiSubmitUrl,JSON.stringify(this.optApply))
            .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
                that.goTo();
              }});
              this.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_MEDICALASSISTANTSUBMIT_APPLY_OK});
            }else {
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "COMMON_ADD_FAIL"){
                this.$dialog.notify({mes: '添加失败', icon: 'error', timeout: 1200});
              }else if(data.code == "NOT_LOOKING_FOR_INFORMATION"){
                this.$dialog.notify({mes: '未查询到信息', icon: 'error', timeout: 1200});
              }else if(data.code == "WEBSITE_QUERY_FAILURE"){
                this.$dialog.notify({mes: '就医网点查询错误码', icon: 'error', timeout: 1200});
              }else{
                this.$dialog.notify({mes: '添加失败', icon: 'error', timeout: 1200});
              }
              this.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_MEDICALASSISTANTSUBMIT_APPLY_FAIL});
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      goTo(){
        this.$router.push({
          path: '/serviceInfo',
          query: {
            code:this.$route.query.code
          }
        });
      }
    }
  }
</script>