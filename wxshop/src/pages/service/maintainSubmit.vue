<template>
	<section class="service">
    <article class="s_contract maintain_sub" style="padding-bottom: 3rem;">
        <div class="i_top">
          <div class="item">
             <span>姓名：</span><input type="text" class="m_w" @focus="handleFocus" v-on:blur="changeInput" maxlength="12" v-model="optApply.contacts"  placeholder="请输入车主姓名"/>
          </div>
          <div class="item">
            <span>手机号：</span><input type="number" class="m_w" v-model="optApply.phone" @focus="handleFocus" v-on:blur="changeInput" maxlength="11" placeholder="请输入车主手机号码" />
          </div>
          <div class="item">
            <span>车架号：</span><input type="text" class="m_w" @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.vinNo" maxlength="50" placeholder="请输入车牌号或车架号后六位" />
          </div>
          <div class="item no_bor maintain_time" v-if="datetime2">
            预约时间：<input type="text" class="m_w" @focus="tiemFocus(2)" placeholder="选择预约时间（请提前24小时）" />
            <i class="time_logo">&nbsp;</i>
          </div>
          <div class="item no_bor maintain_time" v-show="!datetime2">
            <span class="i_t">预约时间：</span><yd-datetime type="date" ref="datetime2" v-model="optApply.beforeTime" slot="right"></yd-datetime><i class="time_logo">&nbsp;</i>
          </div>
        </div>
        <div class="maintain-type">
          <div class="item no_bor s_con" @click="openTip(1)">
            <div><span>预约类型：</span><input type="text" v-model="appointment" disabled readonly="readonly" placeholder="请选择预约类型" />
            </div>
            <div><i class="right">&nbsp;</i></div>
          </div>
        </div>
        <div class="i_bottom">
          <div class="main_addres no_bor s_con">
            <div>车主地址：</br>
              <div class="m_textarea">
                <textarea @focus="handleFocus" v-on:blur="changeInput" v-model="optApply.address" placeholder="请输入车主的地址信息"></textarea>
                <div><i class="i_add" @click="getLocation">&nbsp;</i>获取当前地址</div>
              </div>
            </div>
          </div>
        </div>
        <!-- <div class="s_user_notice">
          <i class="notice_logo">&nbsp;</i>
          <span class="notice_mess">已阅读<span>使用须知</span></span>
        </div> -->
    </article>

    <!--预约类型-->
    <yd-popup v-model="tips1" position="bottom" class="ser-legalfield" height="40%">
      <section class="absol">
        <div>
          <span @click="tipClose(1)">取消</span>
        </div>
        <div>
          <span @click="tipClose(1,'appointment')">确定</span>
        </div>
      </section>  
      <ul class="shop">
        <li v-for="item in optSelect.appointment" @click="selectItem('appointment',item.value,item.name)"><span>{{item.name}}</span><i :class="{'currGou':appointment==item.name,'gou':appointment!=item.name}">&nbsp;</i></li>
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
        title: '保养取送车',
        apiSubmitUrl: window.baseUrl+window.appName+'/shopapi/func/serve/rightsTakeSendCar?shopsid='+window.shopsid,//国际驾照提交信息
        start1:false,
        tips1:false,
        isFileBtn:true,
        datetime2: true,
        isKeyUp:false,
        optSelect:{
          appointment:[{value:1,name:'取送'},{value:2,name:'单取'},{value:3,name:'单送'}],//预约类型
        },
        appointment:'',
        optsAddressData: {
          consignee: '',
          phone: '',
          provinceId: '',
          cityId: '',
          areaId: '',
          address: '',
          defaultBoolean: false
        },
        model2:'',
        beforeTime:'',
        btnShow:true,
        winHeight:0,
        optApply: {
          phone:'',//手机号
          address:'',//车主地址
          contacts: '',//客户姓名
          beforeTime: '',//预约时间
          vinNo: '',//车架号(后6位)
          type: '' //类型: 1取送; 2单取; 3单送;
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
      this.getLocation();
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
        var maintainData = getSessionStore('maintainSubmitInfo',true);
        if(typeof(maintainData) !='boolean') {
          this.optApply = maintainData;
          this.beforeTime = maintainData.beforeTime;
          this.datetime2 = false;
          if(this.optApply.type =='1'){
            this.appointment = '双程';
          }else if(this.optApply.type =='2'){
            this.appointment = '单取';
          }else{
            this.appointment = '单送';
          }
        }
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_MAINTAINSUBMIT_SAVE});
      },
      getLocation() {
          //H5获取当前坐标
          if(navigator.geolocation) {
            document.title = '正在获取当前位置...';
            navigator.geolocation.getCurrentPosition(this.showPosition);
            setTimeout(()=>{
              document.title = this.title;
              this.start1 = true;
            },5000);
          }
      },
      showPosition(position) {
          //百度地图通过坐标转换具体位置
          var that = this;
          var lat = position.coords.latitude;
          var lon = position.coords.longitude;

          MP().then(BMap => {
            var point = new BMap.Point(lon, lat);
            var gc = new BMap.Geocoder();
            gc.getLocation(point, function(rs) {
              document.title = that.title;
              var addComp = rs.addressComponents;
              that.optsAddressData.provinceId = addComp.province;
              that.optsAddressData.cityId = addComp.city;
              that.optsAddressData.areaId = addComp.district;
              that.optsAddressData.address = addComp.province + addComp.city + addComp.district+addComp.street;
              that.model2 = addComp.province + ' ' + addComp.city + ' ' + addComp.district;
              that.start1 = true;
            });
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
      openTip(tips){
        this['tips'+tips] = true;
      },
       //选择类型项
      selectItem(pStr,pItem,pName){
        this.appointment = pName;
        this.optApply.type = pItem;
        // this.optSelect[pStr] = pName;
      },
      tipClose(tips,str){
        this['tips'+tips]=false;
        // if(str){
        //   this.optApply[str] = this[str];
        // }
      },
      //获取当前时间
      getCurDate(){
        var curDate = new Date();
        var y = curDate.getFullYear();
        var m = curDate.getMonth()+1>9?curDate.getMonth()+1:'0'+ (curDate.getMonth()+1);
        var d = curDate.getUTCDate()>9?curDate.getUTCDate():'0'+ curDate.getUTCDate();
        var str = y+'-'+m+'-'+d;
        this.optApply.beforeTime = str;
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
      submit(){
        
        let that = this;
        //提交数据
        if(ydFunLib.isEmptyValid(that.optApply.contacts)){
          that.$dialog.notify({mes: '请输入车主姓名', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.phone)){
          that.$dialog.notify({mes: '请输入手机号码', icon: 'error', timeout: 1200});
          return;
        }else{
          if(!ydFunLib.telInvalid(that.optApply.phone)) {
            that.$dialog.notify({mes: '手机号输入有误', icon: 'error', timeout: 1200});
            return;
          }
        }
        if(ydFunLib.isEmptyValid(that.optApply.vinNo)){
          that.$dialog.notify({mes: '请输入车架号', icon: 'error', timeout: 1200});
          return;
        }
        if(that.beforeTime == ''){
          that.$dialog.notify({mes: '请输入预约时间', icon: 'error', timeout: 1200});
          return;
        }
        if(that.optApply.type == ''){
          that.$dialog.notify({mes: '请选择预约类型', icon: 'error', timeout: 1200});
          return;
        }
        if(ydFunLib.isEmptyValid(that.optApply.address)){
          that.$dialog.notify({mes: '请输入车主地址', icon: 'error', timeout: 1200});
          return;
        }
        console.log(this.optApply);
        this.$dialog.loading.open('保存中...');

        setSessionStore('maintainSubmitInfo',this.optApply);//缓存国际驾照资料确认信息
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.BTN_PAGE_MAINTAINSUBMIT_SAVE});
        this.$http.post(this.apiSubmitUrl,JSON.stringify(this.optApply))
            .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1000 ,callback: () => {
                that.goTo()
              }});
              that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_MAINTAINSUBMIT_APPLY_OK});
            }else {
              if(data.code == "COMMON_ADD_FAIL"){
                that.$dialog.toast({mes: '提交信息在处理中', icon: 'success', timeout: 2000 ,callback: () => {
                  that.goTo();
                }});
              }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      }
    }
  }
</script>