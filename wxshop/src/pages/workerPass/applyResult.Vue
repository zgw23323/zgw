<template>
  <transition name="page">
    <section class="ck-doc applyResult">
      <div>
          <div class="result-head">
            <div class="result-des">
              <p class="ser_type" v-if="serviceType == 'BLBZKD'">玻璃单独保障服务</p>
              <p class="ser_type" v-else-if="serviceType == 'BLBZSP'">玻璃保修服务</p>
              <p class="ser_type" v-else-if="serviceType == 'CZTSTZ'">充值短信通知</p>
              <p class="ser_type" v-else-if="serviceType == 'SPRINGWARM'">春节送温暖</p>
              <p class="ser_type" v-else-if="serviceType == 'APPOINT'">预约维修保养</p>
              <p class="ser_type" v-else>门店财务</p>
              <p class="ser_statu" v-if="serviceValue == '1'">申请中...</p>
              <p class="ser_statu ser_statu1" v-if="serviceValue != '1' && serviceType == '3'">通过微信公众号接收客户充值信息</p>
            </div>
          </div>
          <div class='result-info'>
              <div class='apply-title'>申请信息如下：</div>
              <ul>
                 <li class="item1">                   
                    <div>姓名：</div>
                    <div>{{resultData.name}}</div>
                 </li>
                 <li class="item1">                   
                    <div>手机号：</div>
                    <div>{{resultData.phone}}</div>
                 </li>
                  <li class="item1">                   
                    <div>所属门店：</div>
                    <div style="padding: 0 .1rem 0 0;" @click="selectStore">{{storeName}}<i class="right">&nbsp;</i></div>
                 </li>
                 <li class="item1">                   
                    <div>职位：</div>
                    <div>{{resultData.jobTitle}}</div>
                 </li>
                 <li class="item1">                   
                    <div>申请开通权限：</div>
                    <div v-for="item in serviceTypeData" v-if="item.value == serviceType">{{item.name}}</div>
                 </li>
              </ul> 
          </div>
          <div class="result_btn" v-if="serviceValue == '1'" @click="cancelApply">取消申请</div>
          <div class="result_btn" v-else-if="serviceValue == '2'" @click="saveApply">保存</div>
          <div class="result_btn" v-else @click="updatePushApply">申请开通</div>
      </div>

      <!-- 选择门店 -->
      <div class='tip_f' v-if="tips1" v-model="tips1" @click="tipOpen(1)">&nbsp;</div>
      <div class='ck-tipsFormat center-tip service_cancel_tip' v-model="tips1" v-if="tips1">
          <section class="service_cancel">
            <p>申请绑定的门店</p>
            <div class="service_li max_height" v-if="serviceValue != '1'">
                <div :class="{'has': allChecked}" @click='allCheckBox'>全选</div>
                <div :class="{'has': item.isSelect}" v-for="(item,index) in storeList" @click= "storeBtn(item,index)">{{item.orgShortName}}</div>
            </div>
            <div class="service_li max_height" v-else>
                <div :class="{'has': item.isSelect}" v-if="item.isSelect" v-for="item in storeList">{{item.orgShortName}}</div>
            </div>
            <div class="bottom_btn fiex_tip" v-if="serviceValue != '1'">
              <div @click="tipOpen(1)">取消</div>
              <!-- <div @click="saveApply" v-if="serviceValue == '2'">确定</div> -->
              <div @click="tipOpen(2)">确定</div>
            </div>
            <div class="bottom_btn fiex_tip" v-else>
              <div @click="tipOpen(1)" style="width: 100%;border-right:none;">关闭</div>
            </div>
          </section>  
      </div>
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
      title: '查看申请详情',
      serviceType:'',
      serviceValue:'',
      serviceTypeData:[
        {
          name:'玻璃单独保障服务',
          value:'BLBZKD'
        },
        {
          name:'玻璃保修服务',
          value:'BLBZSP'
        },
        {
          name:'短信推送通知',
          value:'CZTSTZ'
        },
        {
          name:'门店财务',
          value:'MDCW'
        },
        {
          name:'春节送温暖',
          value:'SPRINGWARM'
        },
        {
          name:'预约维修保养',
          value: 'APPOINT'
        }
      ],
      storeSelect:[],
      storeName:'',
      apiServerDetailUrl: window.baseUrl+window.appName+'/wxpay/findServerDetail',//服务详情
      apiCancelApplyUrl: window.baseUrl+window.appName+'/wxpay/cannel/pushApply',//取消申请
      apiApplyUrl: window.baseUrl+window.appName+'/wxpay/apply/pushApply',//申请开通
      apiApplyOrgUrl: window.baseUrl+window.appName+'/wxpay/update/applyOrg',//修改门店
      storeApiUrl: window.baseUrl+window.appName+'/wxpay/listOrgs?orgCode='+window.orgCode,//门店列表
      optUpdateApply:{//更新推送状态
        applyId:"",
        serverCode: ""
      },
      allChecked:false,
      storeList:[],
      storeListTmp:[], 
      tips1:false,
     resultData:{
        status:2,
        name:'',
        phone:'',
        id:'',
        logopath:'',
        jobTitle:'',
        orgId:'',
        orgShortName:'',
        orgLists:[],
        servers:[]
      }           
    }
  },
  watch: {
    '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据
      document.title = this.title;
      this.serviceType = this.$route.query.serverCode;
      this.serviceValue = this.$route.query.value;
      this.fetchServiceInfo();

      var s_1 = getSessionStore('s_1',false);
      if(typeof(s_1) !='boolean'){
        removeSessionStore('s_1');
      }
      this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_APPLYRESULT});
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
    tipOpen(n){
      if(n==2){
        if(this.storeSelect.length== 0){
          this.$dialog.notify({mes: '请选择开通的门店', icon: 'error', timeout: 1200,callback: () => {
            return false;
          }});
        }else{
          this.tips1 = !this.tips1;
        }
      }else{
        this.tips1 = !this.tips1;
      }
    },
    cancelApply(){
          //获取个人中心页面数据
       this.tips1 = true;
    },
    selectStore() {
        //跳去地址
        setSessionStore('s_1','11');
        setSessionStore('information_selected_store',this.storeSelect);
        this.$router.push({
          path: '/store',
          query: {
            from: 'applyResult',
            serverCode: this.serviceType,
            serviceValue: this.serviceValue,
            applyId : this.$route.query.applyId,
          }          
        });
    },
    //取消申请
    cancelApply(){
      let that = this;
      that.optUpdateApply.applyId = that.$route.query.applyId;
      that.optUpdateApply.serverCode = that.$route.query.serverCode;

      that.$dialog.loading.open('加载数据中...');
      that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.BTN_PAGE_APPLYRESULT_CANCEL_APPLY});
      that.$http.post(that.apiCancelApplyUrl,that.optUpdateApply)
        .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            this.$dialog.toast({mes: '申请成功', icon: 'success', timeout: 2000 ,callback: () => {
              this.goTo();
            }});
            this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_APPLYRESULT_CANCEL_APPLY_OK});
          }else {
            this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_APPLYRESULT_CANCEL_APPLY_FAIL});
            this.$dialog.notify({mes: "网络异常，请稍后再来解绑~", icon: 'error', timeout: 1000});
          }
        },(response) => {
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    //申请开通
    updatePushApply(str){
      let that = this;
      var len = [];
      for (var i = 0; i < this.storeSelect. length; i++) {
        len.push(this.storeSelect[i].id);
      }
      that.optUpdateApply.applyId = that.$route.query.applyId;
      that.optUpdateApply.serverCode = that.$route.query.serverCode;
      that.optUpdateApply.orgIds = len;

      that.$dialog.loading.open('加载数据中...');
      this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.BTN_PAGE_APPLYRESULT_APPLY});
      that.$http.post(that.apiApplyUrl,that.optUpdateApply)
        .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            this.$dialog.toast({mes: '申请成功', icon: 'success', timeout: 2000 ,callback: () => {
              this.goTo();
            }});
            this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_APPLYRESULT_APPLY_OK});
          }else {
            that.tips1 = true;
            this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_APPLYRESULT_APPLY_FAIL});
            this.$dialog.notify({mes: "网络异常，请稍后再来解绑~", icon: 'error', timeout: 1000});
          }
        },(response) => {
          that.tips1 = true;
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
        });
    },
    goTo(){
      this.$router.push({
        path: '/infoPushApply'
      });
    },
    //保存修改门店
    saveApply(){
      let that = this;
      var len = [];
      for (var i = 0; i < this.storeSelect. length; i++) {
        len.push(this.storeSelect[i].id);
      }
      var opt = {
        applyId: this.$route.query.applyId,
        serverCode: this.$route.query.serverCode,
        applyOrgIds: len,
        serverStatus: parseInt(this.$route.query.value)
      };

      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.apiApplyOrgUrl,opt)
        .then((response) => {
          const data = response.data;
          if (data.code === 'S_OK') {
            that.$dialog.toast({mes: '保存成功', icon: 'success', timeout: 1500 ,callback: () => {
               that.tips1 = false;        
            }});
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
    //门店点击
    storeBtn(pObj,pIndex) {
      this.storeList[pIndex].isSelect = !this.storeList[pIndex].isSelect;
      console.log('storeSelect->',this.storeSelect);
      var len = this.storeList.filter(function(data){
        return data.isSelect == true;
      });
      this.storeSelect = len;
      
      if(this.storeList.length == len.length){
        this.allChecked = true;
      }else{
        this.allChecked = false;
      }
      if(len.length>1){
        this.storeName = this.storeSelect[0].orgShortName+'...';
      }else if(len.length == 1) {
        this.storeName = this.storeSelect[0].orgShortName;
      }else{
        this.storeName = '';
      }
    },
    //全选点击
    allCheckBox(){
      this.allChecked = !this.allChecked;
      this.storeSelect = [];
      let that = this;
      for(var i=0;i<this.storeList.length;i++){
        this.storeList[i].isSelect = this.allChecked;
        if(this.allChecked){
          this.storeSelect.push(this.storeList[i]);
        }          
      }
      if(this.storeSelect.length>1){
        this.storeName = this.storeSelect[0].orgShortName+'...';
      }else if(this.storeSelect == 1) {
        this.storeName = this.storeSelect[0].orgShortName;
      }else{
        this.storeName = '';
      }
    },
    fetchStoreInfo(){
      //获取门店列表请求方法
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
              for(var j=0;j<that.storeSelect.length;j++){
                var _j = that.storeSelect[j];
                if(_i.id==_j.orgId){
                  data.var[i].isSelect = true;
                }
              }             
            }
            that.storeList = data.var;
            var len = that.storeList.filter(function(data){
              return data.isSelect == true;
            });
            if(len.length==that.storeList.length){
              that.allChecked = true;
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
    fetchServiceInfo(){
      //获取服务详情
      let that = this;
      var optApply ={
        applyId : this.$route.query.applyId,
        serverCode: this.$route.query.serverCode
      };
      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.apiServerDetailUrl,optApply)
        .then((response) => {
          const data = response.data;
          if (data.code === 'S_OK') {
            that.resultData = ydFunLib.ydExtend(that.resultData,data.var,false);
            console.log('that.resultData',that.resultData);
            if(this.$route.query.type){
              var storeStr = getSessionStore('information_selected_store',true);
              if(typeof(storeStr) != 'boolean'){
                this.storeSelect = storeStr;
                this.storeName = this.storeSelect[0].orgShortName;
              }
            }else{
              if(that.resultData.orgLists.length>0){
                that.storeName = that.resultData.orgLists[0].orgShortName;
                that.storeSelect = that.resultData.orgLists;
                for(var i=0;i<that.storeSelect.length;i++){
                  that.storeSelect[i].id = that.storeSelect[i].orgId;
                }
              }else{
                var opt = {
                  id: that.resultData.orgId,
                  orgShortName: that.resultData.orgShortName,
                  orgId: that.resultData.orgId
                };
                that.storeName = that.resultData.orgShortName;
                that.storeSelect.push(opt);
              }
            }
            if(that.storeSelect.length>1){
              that.storeName = that.storeName+'...';
            }
            // this.fetchStoreInfo();
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
    tipsCancel(){
      this.tips1 = false;
    }
  }
}
</script>
