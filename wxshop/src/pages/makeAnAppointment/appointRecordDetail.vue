<template>
    <section class="c-doc appointmentSuccess" v-if="intoType == 'appointer'">
      <article class="content_head">
          <div class="h_message">* 亲，您已预约成功，为减少您到店等待时间，请提前10分钟到店噢！因客户较多，超过预约时间的需要重新预约噢！</div>
          <!-- 超出预约时间10分钟外 -->
          <div v-if="appStatu==3">
            <div class="d_1"><i class="warm_logo">&nbsp;</i></div>
            <div class="d_2">亲，预约时间已过，如不能及时到达，<br>请及时与工作人员联系噢。</div>
          </div>
          <!-- 超出预约时间、预约时间 -->
          <div v-else>
            <div class="d_1"><i class="time_logo">&nbsp;</i></div>
            <div class="d_2">
              <p class="p1">距离预约时间还有</p>
              <!--10分钟以内-->
              <p class="p2" v-if="appStatu==1">
                <span class="s_1">00</span><span class="s_2">时</span>
                <span class="s_1">00</span><span class="s_2">分</span>
                <span class="s_1">00</span><span class="s_2">秒</span>
              </p>
              <!--预约时间内-->
              <p class="p2" v-else-if = "appStatu==2">
                <yd-countdown class="s1" ref='countDown' :cursystem="appointmentObj.currentTime" :countType="'2'" :time="baseInfo.appointTime" :callback="countDownCall" :format="acformat"></yd-countdown>
              </p>
            </div>
          </div>
      </article>
      <article class="content_body">
        <div class="item">
          <p class="i_title"><i class="message_logo">&nbsp;</i><span>预约信息</span></p>
          <div class="i_content">
            <p>姓名：{{baseInfo.userName}}</p>
            <p>手机号：{{baseInfo.userPhone}}</p>
            <p>车牌号：{{baseInfo.licensePlate}}</p>
            <p>预约时间：{{baseInfo.appointTime}}</p>
            <p>接待工作人员：{{baseInfo.saName}}</p>
          </div>
        </div>
        <div class="item">
          <p class="i_title"><i class="store_logo">&nbsp;</i><span>预约门店</span></p>
          <div class="i_content">
            <p class="p1">{{baseInfo.orgName}}</p>
            <p class="p2">电话：15203004897</p>
            <p class="p2">地址：深圳市联合华鹏汽车市场经营管理有限公司A栋1-5号</p>
          </div>
        </div>
        <div class="item no_bor">
          <p class="i_title"><i class="service_logo">&nbsp;</i><span>预约项目</span></p>
          <div class="i_content" v-if="appointmentObj.packageList && appointmentObj.packageList.length>0">
            <p class="package_n">保养套餐</p>
            <div class="package_item">
              <ul class="p_ul">
                <li v-for="package,index in appointmentObj.packageList">
                  <!-- <p>1.{{index+1}}、{{package.projectName}}</p> -->
                  <ul class="p_money">
                    <li>{{package.projectName}}<span class="s_3">￥{{package.projectPrice}}</span></li>
                    <!-- <li>更换电池线</li> -->
                  </ul>
                </li>
                <!-- <li>
                  <p>1.2、配件</p>
                  <ul class="p_li p_money">
                    <li><span>左风口</span><span class="s_3">￥120</span></li>
                    <li><span>左行李架分总成</span><span class="s_3">￥120</span></li>
                    <li><span>左倒车雷达孔堵盖</span><span class="s_3">￥120</span></li>
                  </ul>
                </li> -->
              </ul>
              <div class="package_total">
                <div v-if="getProjectTime(appointmentObj.packageList)"><span>预计时长：</span><span class="s_3" v-html="getProjectTime(appointmentObj.packageList)"></span></div>
                <div><span>优惠金额：</span><span class="s_3">￥{{appointmentObj.partTotal}}</span></div>
              </div>
            </div>
          </div>
          <div class="i_content" v-if="appointmentObj.repairList && appointmentObj.repairList.length>0">
            <p class="package_n">维修项目</p>
            <div class="package_item">
              <ul class="p_ul">
                <li>
                    <ul class="p_money">
                      <li v-for="item,index in appointmentObj.repairList"><span>{{item.projectName}}</span><span>￥{{item.projectPrice}}</span></li>
                      <!-- <li><span>左行李架分总成</span><span>￥120</span></li> -->
                      <!-- <li><span>左倒车雷达孔堵盖</span><span>￥120</span></li> -->
                    </ul>
                </li>
              </ul>
              <div class="package_total">
                <div v-if="getProjectTime(appointmentObj.repairList)"><span>预计时长：</span><span class="s_3" v-html="getProjectTime(appointmentObj.repairList)"></span></div>
                <div><span>优惠金额：</span><span class="s_3">￥{{appointmentObj.pkgTotal}}</span></div>
              </div>
            </div>
          </div>
          <div class="i_content" v-if="appointmentObj.partList && appointmentObj.partList.length>0">
            <p class="package_n">配件</p>
            <div class="package_item">
              <ul class="p_ul">
                <li>
                    <ul class="p_money">
                      <li v-for="item,index in appointmentObj.partList"><span>{{item.projectName}}</span><span>￥{{item.projectPrice}}</span></li>
                      <!-- <li><span>左行李架分总成</span><span>￥120</span></li> -->
                      <!-- <li><span>左倒车雷达孔堵盖</span><span>￥120</span></li> -->
                    </ul>
                </li>
              </ul>
              <div class="package_total">
                <div v-if="getProjectTime(appointmentObj.partList)"><span>预计时长：</span><span class="s_3" v-html="getProjectTime(appointmentObj.partList)"></span></div>
                <div><span>优惠金额：</span><span class="s_3">￥{{appointmentObj.partTotal}}</span></div>
              </div>
            </div>
          </div>
        </div>
      </article> 
      <article class="d_fixed">
        <div class="fixed_btn cancel" @click="cancelBtn">取消预约</div>
        <div class="fixed_btn add" v-if="appStatu ==1 ||appStatu ==3 ">联系工作人员</div>
      </article>
    </section>
    <section class="c-doc receiveMessage" v-else-if="intoType =='worker'">
      <img class="notice-img" src="../../img/appointment/notice.png">

      <div class="notice-title" >
        <p v-if="baseInfo.appointStatus == 1">客户已取消汽车维修保养服务</p>
        <p v-else>车维修保养预约通知</p>
        <span>4月24日 16:00</span>
        <span v-if="baseInfo.appointStatus == 1">亲,客户{{baseInfo.userName}}已取消了预约汽车维修保养服务~~</span>
        <span v-else>亲,客户{{baseInfo.userName}}预约汽车维修保养服务~~</span>
      </div>

      <div class="mation">
        <p>预约信息：</p>
        <p>客户姓名：{{baseInfo.userName}}</p>
        <p>手机号：{{baseInfo.userPhone}}</p>
        <p>车牌号：{{baseInfo.licensePlate}}</p>
        <p>预约类别：{{appointType}}</p>
        <p>预约日期：{{baseInfo.appointTime}}</p>
        <p>接待工作人员：{{baseInfo.saName}}</p>
        <span v-if="baseInfo.appointStatus == 1">客户取消预约,可及时与客户沟通!</span>
      </div>

      <!-- <div class="button"><a :href="'tel:'+baseInfo.userPhone">立即沟通</a></div> -->
      <div class="button"><a href='tel:13778332342'>立即沟通</a></div>
      <div class="saveOther" @click="tipOpen" ><span>分配给其它同事</span></div>


      <!--选择审查目的--> 
      <yd-popup v-model="tips4" v-if="tips4" position="bottom" class="ser-legalfield" height="60%">
        <section class="absol">
          <div>
            <span @click="tipClose(4)">取消</span>
          </div>
          <div>
            <span @click="submit">确定</span>
          </div>
        </section>  
        <ul class="shop">
          <li v-for="item in buttList" @click="buttBnt(item)"><span>{{item.saName}}</span><i :class="{'currGou':curButtId==item.id,'gou':curButtId!=item.id}">&nbsp;</i></li>
        </ul>
      </yd-popup>
    </section>
</template>

<script>
import {getStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '预约成功',
        appointmentObj:{},
        tips:false,
        tips4:false,
        baseInfo:{
          userName:'',
          userPhone:'',
          licensePlate:'',
          appointTime:'',
          orgName:'',
          saName:''
        },
        saName:'',
        appointType:'',
        curButtId:'',
        buttList:[],
        intoType:'',
        acformat:'{%d}:{%h}:{%m}:{%s}',
        appStatu:''//当前预约状态
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      var _urlOpt = ydFunLib.getUrlValue();
      this.intoType = _urlOpt.operUser;
      ajaxModel.wxAppointDetailInfo(this,this.fecthData,{openId:_urlOpt.openId,id:_urlOpt.appointId,operUser:_urlOpt.operUser});
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_APPOINTMENTSUCCESS});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
      },
      //设置顶部时间状态
      setTimeStatu(opt){
        var curTime = new Date(opt.currentTime).getTime();//系统当前时间
        var apppintmentTime = new Date(opt.baseInfo.appointTime).getTime();//预约时间
        var overTime = 10*60*1000; //10分钟转成毫秒数
        if(curTime == apppintmentTime || (apppintmentTime+overTime<=curTime)){
          this.appStatu = 1;//超出预约10分钟内
        }else if(curTime < apppintmentTime) {
          this.appStatu = 2;//在预约时间内
        }else {
          this.appStatu = 3;//超出预约时间10分钟
        }
      },
      //打开SA弹窗
      tipOpen(){
        let that = this;
        ajaxModel.querySaListForDate(this,(result)=>{
          that.buttList = result;
          that.tips4 = true;
        },{orgId:this.baseInfo.orgId,dayDate:this.baseInfo.appointTimeString.split(' ')[0]});
      },
      tipClose() {
        this.tips4 = false;
      },
      buttBnt(opt){
        this.curButtId = opt.id;
        this.saName = opt.saName;
      },
      //获取总时长
      getProjectTime(pList){
        var _all = 0;
        for(var i=0;i<pList.length;i++){
          let itemTime = pList[i].projectTime;
          _all = _all + itemTime;
        }
        return _all
      },
      countDownCall() {
        this.appStatu = 1;
      },
      //取消预约
      cancelBtn(){
        let that = this;
        ajaxModel.wxCancleAppoint(this,()=>{
          // console.log('你已取消');
          if( ydFunLib.curNavigator()){
            WeixinJSBridge.call('closeWindow');
          }else{
            that.$router.back();
          }
        },{id:this.baseInfo.id})
      },
      //确定重新分配SA
      submit(){
        if(this.curButtId ==''){
          this.$dialog.notify({mes: '请选择重新分配SA名字', icon: 'error', timeout: 1500});
          return false
        }else{
          let that = this;
          var opt = {appointId:this.baseInfo.id,saId:this.curButtId};
          ajaxModel.wxSaSssignment(this,()=>{
            that.baseInfo.saName = that.saName;
            that.tips4 = false;
          },opt);
        }
      },
      fecthData(result){
        this.appointmentObj = result;
        this.baseInfo = result.baseInfo;
        if(this.baseInfo.isMaintain ==1){
          this.appointType = '汽车保养';
        }
        if(this.baseInfo.isRepair ==1){
          this.appointType = this.appointType == ''? '汽车维修' : this.appointType+'、汽车维修';
        }
        if(this.baseInfo.isSpray ==1){
          this.appointType = this.appointType == ''? '钣喷' : this.appointType+'、钣喷';
        }
        this.setTimeStatu(result);
      }
    }
  }
</script>
