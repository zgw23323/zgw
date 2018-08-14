<template>

  <section class="ck-doc questionDetail">

    <div v-if="!tips1">
      <!-- 问题详情 -->
      <div class="interaction_list questionDetail_info">
        <div class="li1">
          <div class="item_head i_flex_between">
            <div class="item_head_left i_flex">
              <div class="left">
                <img :src="questionDetailData.questionerImg" v-if="questionDetailData.questionerImg" class="img1 img2" />
                <img src="http://image.xiaobaicar.com/default/head/genius.png" v-else class="img1" />
              </div>
              <div class="right">
                  <p>
                    <span>{{questionDetailData.questionerName}}</span>
                    <i class="u_car_logo" :style="{backgroundImage: 'url('+questionDetailData.carLogo + ')'}" v-if="questionDetailData.carLogo">&nbsp;</i>
                    <!-- <span class="g_km">{{questionDetailData.distance}}</span> -->
                  </p>
                  <div class="car_info">
                     <p v-if="questionDetailData.buyTime">购车时间：{{questionDetailData.buyTime.substring(0,10)}}</p>
                     <p>行驶里程：{{questionDetailData.mileage}}km</p>
                  </div>
              </div>
            </div>
            <div class="isEssence" v-if="questionDetailData.isEssence">精华</div>
          </div>
          <div class="item_con">
            <div class="b_2"><span class="q_mess">{{questionDetailData.questionCtx}}<span class="q_people" v-for="item in questionDetailData.inviterName">{{item}}</span></span></div>
            <ul class="question_image i_flex" v-if="questionDetailData.images && questionDetailData.images.length>0">
              <li v-for="imgs in questionDetailData.images">
                <img :src="imgs" v-if="imgs" />
              </li>
            </ul>
            <div v-if="questionDetailData.tags && questionDetailData.tags.length>0">
              <i class="fault_logo">&nbsp;</i>
              <span class="fault_mess" v-for="item in questionDetailData.tags">{{item}}</span>
            </div>
          </div> 
          <div class="item_botton i_flex_between">
            <div class="q_time" v-if="questionDetailData.createTime">{{questionDetailData.createTime.substring(0,10)}}</div>
            <div>
              <span class="s_mess" v-if="questionDetailData.isAdoptedAnswer==1"><i class="s_secess">&nbsp;</i>已解决</span>
              <span class="l_count">{{questionDetailData.answerNum}}条回答</span>
            </div>
          </div>
        </div>
      </div>
      <!-- 回答列表 -->
      <ul class="interaction_list">
        <li :class="{'li1 b_color_13':answer.isAdopted==1,'li1':answer.isAdopted==0}" v-for="answer,index in answerList">
          <div class="item_head i_flex_between">
              <div class="i_flex">
                <img :src="answer.answerImg" class="img1" />
                <div>
                  <span class="i_phone" style="color: #95a7b7;">{{answer.answerName}}</span></br>
                  <span class="T7C4">{{answer.levelName}}</span>
                </div>
                <!-- <span class="g_km">{{answer.distance}}</span> -->
              </div>
              <div class="isEssence adopt_mess" @click="adoptTip(answer,index)" v-if="curUser==1 && answer.isAdopted==0 && questionDetailData.isClosed==0">采纳为最佳</div>
              <div v-if="answer.isAdopted"><i class="isCheac_icon">&nbsp;</i><span class="T5C11">已采纳</span></div>
          </div>
          <div class="item_con answer_con">
            <div class="b_2" @click="toTo(answer)"><span class="q_mess">{{answer.answerCtx}}</span></div>
            <!-- 回答图片列表 -->
            <ul class="answer_image i_flex" v-if="answer.images && answer.images.length>0">
              <li v-for="imgSrc in answer.images">
                <img :src="imgSrc">
              </li>
            </ul>
            <!-- 回答保养建议 -->
            <div class="maintain_advice" v-if="answer.maintance && answer.maintance.length>0">
              <p class="maintain_title">保养建议</p>
              <ul class="advice_list" @click="toTo(answer)">
                <li class="T5C3" v-for="item,index in answer.maintance" v-if="index<4&&!isShowMore ||isShowMore">{{index+1}}.{{item.partName}}</li>
              </ul>
              <div class="maintain__more_btn i_flex_align" v-if="answer.maintance && answer.maintance.length>4" @click="maintanceMoreBtn">
                <span class="T6C6" v-if="!isShowMore">查看更多</span>
                <span class="T6C6" v-else>收起</span>
                <i class="more_icon">&nbsp;</i>
              </div>
            </div>
            <!-- 回答追问 -->
            <div class="zhui" v-if="answer.answerAsk && answer.answerAsk.length>0">
              <div class="zhui_question">
                <ul class="q_mess" @click="toTo(answer,'zhui')">
                  <li v-for="item,index in answer.answerAsk" v-if="index<2" class='i_flex'>
                    <span class="T6C6">{{item.askName}}：</span>
                    <span v-if="!item.isImg" style="max-width:80%;width: 70%;">{{item.askCtx}}</span>
                    <img class="zhui_q_img" v-if="item.isImg" :src="item.askCtx" />
                  </li>
                </ul>
                <div class="maintain__more_btn i_flex_align" v-if="answer.answerAsk && answer.answerAsk.length>2">
                  <router-link :to="{path:'interactionDetail',query:{answerId:answer.id,questionId:questionDetailData.questionId}}" class="T6C6">查看更多</router-link>
                  <router-link :to="{path:'interactionDetail',query:{answerId:answer.id,questionId:questionDetailData.questionId}}" class="more_icon">&nbsp;</router-link>
                </div>
              </div>
            </div>
            <div class="zhui_btn i_flex_align T6C5">
                <div  @click="toTo(answer)"><span class="a_city" v-if="answer.city">{{answer.city}}</span><span>{{answer.createTime.split(' ')[0]}}</span></div>
                <div class="T6C6 i_flex_align_center" v-if="answer.isAdopted==0 && answer.answerMemberId == interactionOpt.memberId">
                  <span class="i_flex_align" @click="answerBtn(answer,2)"><i class="updata_icon icon">&nbsp;</i>修改</span>
                  <span class="i_flex_align" @click='$refs.child.openTip(answer.id)'><i class="delete_icon icon">&nbsp;</i>删除</span>
                </div>
                <div class="T6C6" v-if="answer.isAdopted==1 && curUser==1" @click="addTest(answer)">添加到测评</div>
            </div>
          </div>
        </li>
      </ul>

      <div class="fixed_bottom b_top_10" v-if="(curUser==1 || curUser==3) && questionDetailData.isAdoptedAnswer==0 && questionDetailData.isClosed==0">
        <!-- 游客身份没有回答过此条消息 -->
        <div class="quest_btn b_color_2 T3C7" v-if="curUser==3" @click="answerBtn('',1)"><i class="quest_icon_3">&nbsp;</i>回答</div>

        <!-- 提问者查看问题详情 -->
        <div class="T3C7 i_flex_align" v-else-if="questionDetailData.isAdoptedAnswer==0&&curUser==1 && questionDetailData.isClosed==0 && answerList && answerList.length>0">
          <div class="T3C3" v-if="questionDetailData.isSummon ==0">没有满意的答案？</div>
          <div class="T3C3" v-if="questionDetailData.isSummon ==1">寻找答案中...</div>
          <div class="i_flex">
            <div class="quest_flex_btn b_color_2" v-if="questionDetailData.isSummon ==0" style="margin-right:.3rem;" @click="summonBtn">召唤牛人</div>
            <div class="quest_flex_btn b_color_2" @click="openTip(2)">取消提问</div>
          </div>
        </div>
        <!-- 提问者点击选择满意答案 -->
        <div class="quest_btn b_color_12 T3C7" @click="cancelBtn" v-else-if="curUser==1 && noChect">取消</div>
      </div>
    </div>
    <div class="tipsPage" v-if="tips1">
      <div class="tipsBox">
        <div class="tipStatus" data-type="busy"></div>
        <p>亲，网络出问题啦~~</p>
        <div class="btns" @click="reLoad">重新加载</div>
      </div>
    </div>
    
    <yd-comfig-tip :callback="answerDelete" ref="child" :comfigTip="comfigTip"></yd-comfig-tip>

    <!--选择答案提示-->
    <yd-popup v-model="tips3" position="center" width="80%">
      <section class="question_tip">
        <div class="T3C3"><span>{{adoptTipMess}}</span></div>
        <div class="i_flex_align_center q_t_btn">
          <div class="T3C2" @click="closeTip(3)">否</div>
          <div class="T3C2" @click="adoptBtn">是</div>
        </div>
      </section>  
    </yd-popup>

    <!--选择答案提示-->
    <yd-popup v-model="tips2" position="center" width="80%">
      <section class="question_tip">
        <div class="T3C3"><span class="T2C3">提示</span></br><span class="T3C4">是否要取消提问？</span></div>
        <div class="i_flex_align_center q_t_btn">
          <div class="T3C2" @click="closeTip(2)">取消</div>
          <div class="T3C2" @click="cancelQuestion">确定</div>
        </div>
      </section>  
    </yd-popup>
  </section>

</template>

<script type="text/babel">
  import {setShopsId,setSessionStore,removeSessionStore,getSessionStore,setStore,getStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        ok: false,
        tips1: false,
        tips2:false,
        tips3:false,
        isSingle: false,
        start1: false,
        noChect:false,//提问者是否点击选择答案
        isShowMore:false,//是否显示更多
        title: '问答详情',
        imgBaseUrl: window.imgBaseUrl,
        interactionOpt:{
          memberId:'',
          token:''
        },
        comfigTip:{
          'title': '请确定删除此回答？',
          'mess': '删除后不可恢复！'
        },
        adoptTipMess:"",
        adoptTipId:'',
        cancelId:'',
        apiUrl: window.interactionBaseUrl+'/question/detail?token=',//获取问题详情
        answerUrl: window.interactionBaseUrl+'/answer/list?token=',//获取回答列表
        curUserUrl: window.interactionBaseUrl +'/question/identity?token=',//查看当前身份
        aplAdoptUrl: window.interactionBaseUrl +'/answer/adopt?token=',//采纳答案
        aplDeleteUrl: window.interactionBaseUrl +'/answer/delete?token=',//删除回答
        aplAddMaintancePartUrl: window.interactionBaseUrl +'/answer/addMaintancePart?token=',//添加到测评
        aplSummonUrl: window.interactionBaseUrl +"/question/summon?token=",//召唤牛人
        aplCloseUrl: window.interactionBaseUrl +"/question/close?token=",//取消提问
        opts: {},
        pageNumber:0,
        isMax:false,
        answerOpts:'',
        curUser:'',
        answerList:[],//回答列表
        questionDetailData: {}
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      var _interactionData = getStore('interactionData',true);
      if(typeof(_interactionData) != 'boolean'){
        this.interactionOpt = _interactionData;
        this.getQuestionDetail();
        this.getAnswerData();
      }else{
        this.getLoginData(0);
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
        window.addEventListener('scroll', this.menu);
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_QUESTIONDEATIL});
      },
      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
            this.loadList();
        }
      },
      //登录
      getLoginData(n) {
        var that = this;
        var _type = n;
        ydFunLib.interactionLogin(function(data){
          console.log('data',data);
            that.interactionOpt.memberId = data.memberId;
            that.interactionOpt.token = data.token;
            setStore('interactionData',that.interactionOpt);
            if(_type==0){
              that.getQuestionDetail();
              that.getAnswerData();
            }
        })
      },
      //保养建议查看更多
      maintanceMoreBtn(){
        this.isShowMore = !this.isShowMore;
      },
      //采纳答案确认
      adoptTip(opt,n){
        this.adoptTipMess = '是否采纳'+opt.answerName+'的答案？';
        this.adoptTipId= opt.id;
        this.tips3 = true;
      },
      //取消选择
      cancelBtn(){
        this.noChect = false;
      },
      closeTip(n) {
        this['tips'+n] = false;
      },
      openTip(n) {
        this['tips'+n] = true;
      },
      //跳去追问详情
      toTo(opt,str){
        if(this.curUser==1 || str){
          this.$router.push({ 
            path: '/interactionDetail',
            query: {answerId: opt.id,questionId:this.questionDetailData.id}
          });
        }
      },
      //添加到测评
      addTest(opt) {
        let that = this;
        var _url = this.interactionOpt.token + '&memberId=' + this.interactionOpt.memberId +'&answerId='+opt.id+'&questionId='+this.$route.query.pid;
        var _partId = [];
        var _partName = [];
        for(var i=0;i<opt.maintance.length;i++){
          var item = opt.maintance[i];
          _partId.push(item.partId);
          _partName.push(item.partName);
        }
        _url = _url+'&partId='+_partId.join(',')+"&partName="+_partName.join(',');
        that.$http.get(this.aplAddMaintancePartUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              // this.answerAsList[n].isAdopted = 1;
              that.$dialog.notify({mes: data.code, icon: 'success', timeout: 1200});
            }else if(data.msg=='unauth'){
              console.log('data-msg',data.msg);
              that.getLoginData();
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
            
          },(response) => {
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //召唤牛人
      summonBtn() {
        let that = this;
        var _url = this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId+'&questionId='+this.$route.query.pid;
        that.$dialog.loading.open('加载中...');
        that.$http.get(that.aplSummonUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
              // that.$dialog.toast({mes: '牛人正在火速赶来，请稍后再来采纳答案哦！', icon: 'success', timeout: 5000 ,callback: () => {
              // }});
              that.$dialog.toast({mes: '牛人正在火速赶来，请稍后再来采纳答案哦！', icon: 'success', timeout: 5000});
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      //回答及修改回答点击
      answerBtn (n1,n2){
        var opt = {
          questionCtx: this.questionDetailData.questionCtx,
          questionId:this.questionDetailData.id,
          memberId:this.interactionOpt.memberId,
          token: this.interactionOpt.token
        }
        if(n1){
          opt.answerCtx = n1.answerCtx;
          opt.maintance = n1.maintance;
          opt.images = n1.images;
          opt.id = n1.id;
        }
        setSessionStore('questionDetailData',opt);
        this.$router.push({
          path: '/answerQuestionOne',
          query: {type:n2}
        });
      },
      //删除回答
      answerDelete(pId) {
        let that = this;
        console.log('pId=>',pId);
        var _url = this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId+'&answerId='+pId;
        that.$dialog.loading.open('删除中...');
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.BTN_PAGE_QUESTIONDEATIL_DELETE_APPLY});
        that.$http.get(that.aplDeleteUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
              removeSessionStore('questionDetailData');
              that.$router.go(0);
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_QUESTIONDEATIL_DELETE_APPLY_OK});
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_QUESTIONDEATIL_DELETE_APPLY_FAIL});
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_QUESTIONDEATIL_DELETE_APPLY_FAIL});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      //取消提问
      cancelQuestion() {
        let that = this;
        var _url = this.interactionOpt.token + '&memberId=' + this.interactionOpt.memberId +'&questionId='+this.$route.query.pid;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.aplCloseUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              that.$dialog.toast({mes: '取消成功', icon: 'success', timeout: 2000 ,callback: () => {
                this.$router.push({ 
                  path: '/interactionHome'
                });
              }});
            }else {
              console.log(data.msg);
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
            
          },(response) => {
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //采纳问题
      adoptBtn(){
        var that = this;
        var _id= this.$route.query.answerId;
        var _url = this.interactionOpt.token + '&memberId=' + this.interactionOpt.memberId +'&answerId='+this.adoptTipId+'&questionId='+this.$route.query.pid;
        this.opts = _url;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.aplAdoptUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              for(var i=0;i<that.answerList.length;i++){
                if(that.answerList[i].id==that.adoptTipId){
                  that.answerList[i].isAdopted = 1;
                  that.questionDetailData.isAdoptedAnswer = 1;
                  break;
                }
              }
            }else {
              console.log(data.msg);
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
            
          },(response) => {
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //查看当前身份
      getCurUser(strUrl){
         let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.get(that.curUserUrl+strUrl)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "s_ok") {
              that.curUser = data.result;//1:提问者 2：回答者 3：第三方用户 4:问题已删除
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      loadList() {
        //分页加载异步请求数据
        let that = this;
        that.pageNumber++;
        if(this.isMax) {
          /* 所有数据加载完毕 */
          that.isLoading = false;
          // that.isDone = true;
          // that.$yduiBus.$emit('ydui.infinitescroll.loadedDone');
          return;
        }
        that.$http.get(that.answerUrl+that.answerOpts+'&pageNumber='+that.pageNumber)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
              if(data.result){
                for(var i=0;i<data.result.length;i++){
                  var item = data.result[i];
                  // data.result.inviterName = item.inviterName ? item.inviterName.split(','):[];
                  data.result[i].images = item.images ? item.images.split(','):[];
                  data.result[i].maintance = data.result[i].maintance? data.result[i].maintance:[];
                  if(item.answerAsk){
                    for(var j=0;j<item.answerAsk.length;j++){
                      var _item = item.answerAsk[j];
                      if(_item.askCtx.split('http:').length>1){
                        data.result[i].answerAsk[j].isImg=true;
                      }else{
                        data.result[i].answerAsk[j].isImg=false;
                      }
                    }
                  }
                  that.answerList.push(data.result[i]);
                }
                that.isMax = false;
              }else{
                that.isMax = true;
              }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      getAnswerData: function() {
         //获取回答数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
         var _url =  this.interactionOpt.token+'&questionId='+this.$route.query.pid+'&memberId='+this.interactionOpt.memberId;
         this.answerOpts = _url;
        that.$http.get(that.answerUrl+that.answerOpts+'&pageNumber='+that.pageNumber)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
              var answerArr = [];
              if(data.result){
                for(var i=0;i<data.result.length;i++){
                  var item = data.result[i];
                  // data.result.inviterName = item.inviterName ? item.inviterName.split(','):[];
                  data.result[i].images = item.images ? item.images.split(','):[];
                  data.result[i].maintance = data.result[i].maintance? data.result[i].maintance:[];

                  if(item.answerAsk){
                    for(var j=0;j<item.answerAsk.length;j++){
                      var _item = item.answerAsk[j];
                      if(_item.askCtx.split('http:').length>1){
                        data.result[i].answerAsk[j].isImg=true;
                      }else{
                        data.result[i].answerAsk[j].isImg=false;
                      }
                    }
                  }
                  answerArr.push(data.result[i]);
                }
                that.answerList = answerArr;
                that.isMax = false;
              }else{
                that.isMax = true;
              }
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      getQuestionDetail: function() {
        //获取问题详情数据
        let that = this;
        var _url =  this.interactionOpt.token+'&questionId='+this.$route.query.pid+'&memberId='+this.interactionOpt.memberId;
        that.$dialog.loading.open('加载数据中...');
        that.$http.get(that.apiUrl+_url)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "s_ok") {
              var item = data.result;
              data.result.tags = item.tags? item.tags.split(','):[];
              data.result.inviterName = item.inviterName? item.inviterName.split(','):[];
              data.result.images = item.images ? item.images.split(','):[];
              data.result.carLogo = item.carLogo && item.carLogo.split('http:').length>1?item.carLogo:'';
              that.questionDetailData = data.result;
              console.log('that.questionDetailData>=',that.questionDetailData);
              that.getCurUser(_url);
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>

