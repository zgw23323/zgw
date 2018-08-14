<template>
  <transition name="page">
    <section class="worderInfo applyResult">
        <div class="worderInfo_bd">
          <div class="i_item" style="height: 1.39rem;position: relative;">
              <span>头像：</span>
              <div class="u-info">
                <div class="u-head" v-if="personData.logopath !='' && personData.logopath !=null" :style="{backgroundImage: 'url(' + personData.logopath + ')'}"></div>
                <div class="u-head" v-else :style="{backgroundImage: 'url(' + uhead + ')'}">头像</div>
              </div>
          </div>
          <div class="i_item">
            <span>姓名：</span>
            <span>{{personData.name}}</span>
          </div>
          <div class="i_item">
                <span>手机号码：</span>
                <span>{{personData.phone}}</span>
          </div>
          <div class="i_item">
                <span>门店</span>
                <span>{{personData.orgShortName}}</span>
          </div>
          <div class="i_item">
                <span>职位：</span>
                <span>{{personData.jobTitle}}</span>
          </div>
          <div class="i_item" @click="goTo(personData.rechargeCodeUrl)" v-if="personData.rechargeCodeUrl">
                <span>二维码：</span>
                <span class="code_icon">&nbsp;</span>
          </div>
          <div class="item1">
            <span>开通的权限：</span>
            <div>
              <span v-for="(item,index) in serviceList">{{item.name}}<em v-if="index!=openLen-1 && index <= openLen">、</em></span>
            </div>
          </div>
        </div>
        <div class="info_code worker_savebtn" @click="tipOpen(1)" v-if="!personData.rechargeCodeUrl">生成充值推荐人二维码</div>

        <!-- 选择会员系统个人信息 -->
      <div class='tip_f' v-if="tips1" v-model="tips1" @click="tipOpen">&nbsp;</div>
      <div class='ck-tipsFormat center-tip service_cancel_tip' v-model="tips1" v-if="tips1">
          <section class="service_cancel">
            <div class="t_top"><span>请选择您会员系统个人信息</span></br><span>（如未找到您资料，请联系本店会员管理人员）</span></div>
            <ul class="service_li max_height t_content">
              <li v-for='item,index in referrerList' @click="btn(item)"><span>{{item.realName}}</span><i :class="{'t_icon':opt.refereeId != item.id,'gou_icon':opt.refereeId == item.id}">&nbsp;</i></li>
            </ul>
            <div class="bottom_btn fiex_tip">
              <div @click="tipOpen">取消</div>
              <div @click="subBtn">确定</div>
            </div>
          </section>  
      </div>
    </section>
  </transition>
</template>

<script type="text/babel">
import {setShopsId,getSessionStore,setSessionStore,setStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import ajaxModel from '../../utils/ajaxModel';
export default {
    data() {
        return {
          title: '权限服务',
          imgBaseUrl: window.imgBaseUrl,
          tips1:false,
          uhead:'',
          serviceList:[],
          serviceTypeData:{'BLBZKD':'玻璃单独保障服务','BLBZSP':'玻璃保修服务','CZTSTZ':'短信推送通知','MDCW':'门店财务','SPRINGWARM':'春节送温暖活动'},
          openLen:0,//开通的服务
          referrerApiUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxListEmployee',
          //apiApplyUrl: window.baseUrl+window.appName+'/shopapi/func/pushapply/wxGenRechargeCodeUrl?orgCode='+window.orgCode,
          apiApplyUrl: window.baseUrl+window.appName+'/shopapi/func/pushapply/wxGenRechargeCodeUrl',
          referrerList: [],
          opt: {
            pushapplyId:'',
            refereeId:''
          },
          realName:'',
          personData: {}
        }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        //获取缓存个人数据
        var _personData = getSessionStore("workerInfo",true);
        if(typeof(_personData) !='boolean'){
          this.personData = _personData;
          for(var i=0;i<_personData.servers.length;i++){
            var item = _personData.servers[i];
            if(item.serverStatus=='2'||item.serverStatus=='1'){
              var opt = {};
              opt.name = this.serviceTypeData[item.serverCode];
              opt.value =  item.serverStatus;
              this.serviceList.push(opt);
            }
          }
          this.openLen = this.serviceList.length;
        }
 
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_WORKERINFO});
      },
      fetchReferrerList(resultArr) {
        this.referrerList = resultArr;
        this.tips1 = !this.tips1;
      },
      btn(opt) {
        this.opt.refereeId = opt.id;
        this.realName = opt.realName;
      },
      goTo(url){
        setSessionStore('code_info_url',url);
        this.$router.push({path: '/infoCode'});
      },
      subBtn(){
        let that = this;
        if(this.opt.refereeId==''){
          this.$dialog.notify({mes: '请选择您会员系统个人信息', icon: 'error', timeout: 1200});
          return;
        }
        this.opt.pushapplyId = this.personData.id;

        this.$dialog.loading.open('正在生成中...');
        this.$http.post(this.apiApplyUrl+"?orgCode=" + this.personData.orgCode, this.opt)
          .then((response) => {
            const data = response.data;
            this.$dialog.loading.close();
            if(data.code == "S_OK") {
              setStore('infoCode_refereeOpt',{id:that.opt.refereeId,realName:that.realName});//缓存生成二维码
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_INFOCODE_MESS_SUBMIT_OK});
              this.goTo(data.var);
            }else {
              that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_INFOCODE_MESS_SUBMIT_FAIL});
              that.$dialog.notify({mes: "操作失败,请稍后再试", icon: 'error', timeout: 1200});
            }
          },(response) => {
            setTimeout(()=>{
              this.$dialog.loading.close();
              this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      tipOpen() {
        ajaxModel.wxListEmployee(this,this.fetchReferrerList,this.personData.orgId);
      }
    }
}
</script>
