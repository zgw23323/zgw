<template>
    <section class="c-doc appointmentTime">
      <article class="data_head t_fixed">
        <ul class="i_flex" id="top_day">
          <li :class="{'d_cur d_item': isBtn==item.number,'d_item':isBtn != item.number}" v-for="item,index in dateList" v-if="item.number !=''&& ((dateOpt.nowYear == btnDateOpt.nowYear&&dateOpt.nowMonth == btnDateOpt.nowMonth&&item.number>=dateOpt.nowDay)||(dateOpt.nowYear != btnDateOpt.nowYear)||(dateOpt.nowYear == btnDateOpt.nowYear&&dateOpt.nowMonth != btnDateOpt.nowMonth))" @click="dayBtn(item,'top')" :id="item.month+'_'+item.number">
            <p>{{item.month}}-{{item.number}}</p>
            <p>{{item.week}}</p>
          </li>
        </ul>
        <div class="d_icon" @click="tips1 = !tips1"><i class="d_logo">&nbsp;</i></div>
      </article>
      <article class="data_content">
        <ul class="i_flex">
          <li :class="{'d_cur':isBtnTime==item.workTimeString.split(' ')[1],'no_can':item.isOpen==2}" v-for="item in timeList" @click="timeBtn(item)">{{item.workTimeString.split(' ')[1]}}</li>
        </ul>
      </article>
      <div class="m-popup-mask" v-show="tips1">&nbsp;</div>
      <article class="time_tip t_fixed" v-show="tips1">
        <div class="week">
          <span>日</span>
          <span>一</span>
          <span>二</span>
          <span>三</span>
          <span>四</span>
          <span>五</span>
          <span>六</span>
        </div>
        <div class="date">
          <div class="data_title i_flex_between">
            <div class="left" @click="lastMounth"><i class="l_icon">&nbsp;</i></div>
            <div class="center">{{btnDateOpt.nowYear}}年{{btnDateOpt.nowMonth}}月</div>
            <div class="right" @click="mextMouth"><i class="r_icon">&nbsp;</i></div>
          </div>
        </div>
        <ul class="date_li i_flex">
          <li v-for="item,index in dateList" :class="{'d_cur':item.number===isBtn,'n_cur':item.isNow==1,'u_cur':item.number<dateOpt.nowDay && dateOpt.nowYear == btnDateOpt.nowYear && dateOpt.nowMonth == btnDateOpt.nowMonth }"  @click="dayBtn(item,'bottom')">
            <span v-if="item.isNow==1">今天</span>
            <span v-else-if="item.isNow ==2">明天</span>
            <span v-else-if="item.isNow ==3">后天</span>
            <span v-else-if ="item.number !=0">{{item.number}}</span>
          </li>
        </ul>
      </article>
    </section>
</template>

<script>
import {getStore,setSessionStore,getSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '预约时间',
        tips1: false,
        isBtn:0,
        isBtnTime:'',
        curMounth:'',
        dateOpt:{},
        isDefault: false,//是否有默认日期
        btnDateOpt:{},
        opt:{orgId:'',workDate:''},
        timeList:[],
        // timeList:[{isOpen:1,workTimeString:'2018-05-17 08:30'},{isOpen:2,workTimeString:'2018-05-17 09:30'}],
        dateList:[]
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      document.title = this.title;
      this.markInit();
      this.initDate();
      // document.getElementById("top_day").addEventListener('scroll',this.winScroll,false);
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_APPOINTMENTTime});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        var optAppointment = getSessionStore('appointment_message',true);
        if(typeof(optAppointment) != 'boolean'){
          this.opt.orgId = optAppointment.orgId; 
        }
      },
      initDate() {
        this.dateOpt = ydFunLib.getCurDate();
        var strTime = getSessionStore('appointment_time');
        if(typeof(strTime) != 'boolean') {
          var _data = strTime.split(':')[0].split('-');
          this.btnDateOpt.nowYear = Number.parseInt(_data[0]);
          this.btnDateOpt.nowMonth = Number.parseInt(_data[1]);
          this.btnDateOpt.nowDay = Number.parseInt(_data[2]);
          this.isDefault = true;
          this.isBtn = this.btnDateOpt.nowDay;
          this.opt.workDate = this.getAllDate();
          this.isBtnTime = strTime.split(' ')[1];
          ajaxModel.wxWorktimeForOrg(this,this.fecthTime,this.opt);
        }else{
          this.btnDateOpt = ydFunLib.getCurDate();
        }
        var dateNum = ydFunLib.getThisMonthDay(this.btnDateOpt.nowYear,this.btnDateOpt.nowMonth);
        this.setDayList(dateNum,'init');
      },
      getAllDate(){
        var _month = this.btnDateOpt.nowMonth>9? this.btnDateOpt.nowMonth:'0'+this.btnDateOpt.nowMonth;
        var _day = this.btnDateOpt.nowDay>9? this.btnDateOpt.nowDay:'0'+this.btnDateOpt.nowDay;//当前点击日期
        var _date = this.btnDateOpt.nowYear+'-'+_month+'-'+_day;
        return _date
      },
      //获取门店排班时刻表
      fecthTime(result){
        this.timeList = result;
      },
      //日期点击
      dayBtn(obj,str) {
        if(this.dateOpt.nowYear == this.btnDateOpt.nowYear&&this.dateOpt.nowMonth == this.btnDateOpt.nowMonth&&obj.number<=this.dateOpt.nowDay){
          return false;
        }
        this.isBtn = obj.number;
        this.btnDateOpt.nowDay = Number.parseInt(obj.number);
        if(str == 'bottom'){
          this.goToScrollLeft();
          this.tips1=false;
        }
        this.opt.workDate = this.getAllDate();
        ajaxModel.wxWorktimeForOrg(this,this.fecthTime,this.opt);
      },
      //时刻点击
      timeBtn(str){
        if(str.isOpen==2){
          return false
        }
        this.isBtnTime = str.workTimeString.split(' ')[1];
        let _month = this.btnDateOpt.nowMonth>9?this.btnDateOpt.nowMonth:'0'+this.btnDateOpt.nowMonth;
        let _dat = this.btnDateOpt.nowDay>9?this.btnDateOpt.nowDay:'0'+this.btnDateOpt.nowDay;
        setSessionStore('appointment_time',this.btnDateOpt.nowYear+'-'+_month+'-'+_dat+' '+this.isBtnTime);
        this.$router.push({path: '/appointmentService'});
      },
      //设置天数
      setDayList(n,str){
        var monthDay = n;
        this.dateList = [];
        if(str == 'init' && !this.isDefault){//初始化数据
          if(this.dateOpt.nowDay == n){//如果今天是当月的最后一天，默认显示下个月的天数
            var opt = {};
            opt.number = this.dateOpt.nowDay;
            opt.month = this.dateOpt.nowMonth;
            opt.year = this.dateOpt.nowYear;
            opt.week = ydFunLib.getWeek(this.dateOpt.nowYear,this.dateOpt.nowMonth,this.dateOpt.nowDay);
            // console.log('opt.week',opt);
            this.isBtn = 1;
            this.dateList.push(opt);
            if(this.dateOpt.nowYear == 12){//当前月份为12月份
              monthDay = ydFunLib.getThisMonthDay(this.dateOpt.nowYear+1,1);
              this.btnDateOpt.nowYear = this.dateOpt.nowYear+1;
              this.btnDateOpt.nowMonth = 1;
              this.btnDateOpt.nowDay = 1;
            }else{
              monthDay = ydFunLib.getThisMonthDay(this.dateOpt.nowYear,this.dateOpt.nowMonth+1);
              // console.log('monthDay',monthDay);
              this.btnDateOpt.nowMonth = this.dateOpt.nowMonth+1;
              this.btnDateOpt.nowDay = 1;
            }
          }else{//设置默认选中
            this.isBtn = this.dateOpt.nowDay+1;
            this.btnDateOpt.nowDay = this.isBtn;
          }
          this.opt.workDate = this.getAllDate();
          ajaxModel.wxWorktimeForOrg(this,this.fecthTime,this.opt);
        }
        var firstWeek =  ydFunLib.getFirstWeek(this.btnDateOpt.nowYear+'-'+this.btnDateOpt.nowMonth);
        // console.log('firstWeek',firstWeek);
        firstWeek = this.dateList.length ? firstWeek-1:firstWeek;
        //建立空数据填充空白日期
        var emthLen = [];
        for(var j=0;j<firstWeek;j++){
          var opt = {};
          opt.number = '';
          emthLen.push(opt);
        }
        this.dateList = [...emthLen,...this.dateList];
        // console.log('this.dateList11',this.dateList);
        //构造月份日期数
        for(var i=1;i< monthDay+1;i++){
          var opt = {};
          opt.number = i;
          opt.month = this.btnDateOpt.nowMonth;
          opt.year = this.btnDateOpt.nowYear;
          opt.week = ydFunLib.getWeek(this.btnDateOpt.nowYear,this.btnDateOpt.nowMonth,i)
          if(this.btnDateOpt.nowMonth == this.dateOpt.nowMonth){
            if(i==this.dateOpt.nowDay){
              opt.isNow = 1;//今天
            }else if(i == this.dateOpt.nowDay+1){
              opt.isNow = 2;//明天
            }else if(i == this.dateOpt.nowDay+2){
              opt.isNow = 3;//后天
            }
          }
          this.dateList.push(opt);
        }
        // console.log('this.dateList22',this.dateList);
        setTimeout(()=>{
          if(this.isDefault){
            this.goToScrollLeft();
          }else{
            document.getElementById("top_day").scrollLeft = 0;
          }
        },100);
      },
      //下个月点击
      mextMouth() {
        this.isBtn = 0;
        var n = this.btnDateOpt.nowMonth+1;
        this.btnDateOpt.nowMonth = n>12 ? 1 : n;
        this.btnDateOpt.nowYear = n>12 ? this.btnDateOpt.nowYear+1 :this.btnDateOpt.nowYear;
        var dateNum = ydFunLib.getThisMonthDay(this.btnDateOpt.nowYear,this.btnDateOpt.nowMonth);
        this.isDefault = false;
        this.setDayList(dateNum);
      },
      //上个月点击
      lastMounth() {
        if((this.dateOpt.nowYear == this.btnDateOpt.nowYear&&this.dateOpt.nowMonth != this.btnDateOpt.nowMonth) || this.dateOpt.nowYear != this.btnDateOpt.nowYear){
          this.isBtn = 0;
          var n = this.btnDateOpt.nowMonth-1;
          this.btnDateOpt.nowMonth = n==0 ? 12 : n;
          this.btnDateOpt.nowYear = n==0 ? this.btnDateOpt.nowYear-1 :this.btnDateOpt.nowYear;
          var dateNum = ydFunLib.getThisMonthDay(this.btnDateOpt.nowYear,this.btnDateOpt.nowMonth);
          this.isDefault = false;
          this.setDayList(dateNum);
        }
      },
      //跳到指定位置
      goToScrollLeft(){
         var itemSpan = document.getElementsByClassName('d_item');
        //拿到当前屏幕下一个日期标签的宽度
        // console.log('itemSpan[itemSpan.length-1]',itemSpan.length);
        var spanWidth = itemSpan[itemSpan.length-1].clientWidth;
        //拿到当前屏幕两个日期标签距离屏幕左 边的距离差
        var spanOffetWidth = document.getElementsByClassName('d_item')[1].offsetLeft -document.getElementsByClassName('d_item')[0].offsetLeft;
        var itemDay = document.getElementById(Number.parseInt(this.btnDateOpt.nowMonth)+'_'+Number.parseInt(this.isBtn)).offsetLeft;
        document.getElementById("top_day").scrollLeft = itemDay-spanWidth-spanOffetWidth;
      }
    }
  }
</script>
