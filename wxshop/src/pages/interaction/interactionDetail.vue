<template>
  <section class="ck-doc interactionDetail T3C3">
    <article class="part_content">
      <ul>
        <li v-for="item,index in answerAsList" v-if="index==0">
          <div class="a_time"><span class="T6C5" v-if="item.createTime">{{item.createTime.split(' ')[0]}}</span></div>
          <div class="i_flex">
            <div><i class="u_logo" :style="{backgroundImage: 'url(' + item.askImg + ')'}">&nbsp;</i></div>
              <div class="a_logo answerAsk_text">
                <span class="T3C6">问题：</span>{{item.questionCtx}}
              </div>
          </div>
        </li>
        <li v-for="item,index in answerAsList">
          <div class="a_time"><span class="T6C5" v-if="item.createTime">{{item.createTime.split(' ')[0]}}</span></div>
          <div v-if="item.memberId !=item.questionerId" class="i_flex_justify_end">
            <div class="a_logo answerAsk_text" v-if="item.askCtx">
              <!-- <span v-if="index==0"><span class="T3C2">问题：</span>{{item.askCtx}}</span> -->
               <span>{{item.askCtx}}</span>
            </div>
            <div class="a_logo answerAsk_img" v-if="item.images">
              <img :src="item.images" />
            </div>
            <div><i class="u_logo" :style="{backgroundImage: 'url(' + item.askImg + ')'}" style="margin-left:.3rem;">&nbsp;</i></div>
            <!-- <div><i class="u_logo" style="margin-left:.3rem;">&nbsp;</i></div> -->
          </div>
          <div v-else class="i_flex">
            <div><i class="u_logo" :style="{backgroundImage: 'url(' + item.askImg + ')'}">&nbsp;</i></div>
              <!-- <div ><i class="u_logo">&nbsp;</i></div> -->
              <div class="a_logo answerAsk_text" v-if="item.askCtx">
                <span v-html="item.askCtx"></span>
              </div>
              <div class="answerAsk_img" v-if="item.images">
                <img :src="item.images" />
              </div>
          </div>
        </li>
      </ul>
    </article>
    <div class="fixed_bottom T3C2 i_flex_align_center" v-if="curUser==1||curUser==2">
      <div class="item">
        <form id="input-file" enctype="multipart/form-data" :action="apiUploadFileUrl" method="post" target="uploadIframe">
          <input type="text" name="token" v-show="false" id="token" :value="interactionOpt.token">
          <i class="p_logo">&nbsp;</i>
          <input type="file" class="up_file_input" name="upload" @change="readFile" id="input_file" accept="image/*" />
        </form>
      </div>
        <input class="in_text" v-model="answerText" type="text"/>
        <span class="sen_btn T3C7" @click="saveSubmit('text')">完成</span>
      <!-- </div> -->
    </div>
    <yd-imgLoading-tip v-if="tips4"></yd-imgLoading-tip>
  </section>

</template>

<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,getStore,setStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '互动详情',
        tips4:false,
        tagCount:0,
        imgList:'',
        apiAnswerAskUrl: window.interactionBaseUrl+'/answerAsk/list?token=',//获取追问列表
        apiUploadFileUrl: window.interactionBaseUrl+'/upload/uploadImg',
        curUserUrl: window.interactionBaseUrl +'/question/identity?token=',//查看当前身份
        apiAnswerAddUrl: window.interactionBaseUrl+'/answerAsk/add?token=',//添加追问
        interactionOpt:{
          memberId:'1',
          token:'',
          mobile:'123456789009'
        },
        opts:'',
        curUser:'',
        pageNumber:0,
        answerText:'',
        qType:'',
        answerAsList:[],
        applyOpt:{}
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由0.005rem助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      
      var _interactionData = getStore('interactionData',true);
      if(typeof(_interactionData) != 'boolean'){
        this.interactionOpt = _interactionData;
      }
      this.getAnswerAskData();
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_INTERACTIONDETAIL});
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
            if(_type==1){
              that.getAnswerAskData();
            }else if(_type==2) {
              that.saveSubmit(that.qType);
            }
        })
      },
      //查看当前身份
      getCurUser(){
         let that = this;
         var _url =  this.interactionOpt.token+'&questionId='+this.$route.query.questionId+'&memberId='+this.interactionOpt.memberId;
        that.$dialog.loading.open('加载数据中...');
        that.$http.get(that.curUserUrl+_url)
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
      //完成
      saveSubmit(str) {
        let that = this;
        this.qType = str;
        var _url = this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId+'&answerId='+this.$route.query.answerId+'&questionId='+this.$route.query.questionId;
        if(str == 'text'){
          if(this.answerText==''){
            this.$dialog.notify({mes: '追加内容不能为空', icon: 'error', timeout: 1200});
            return;
          }else if(this.answerText.length<12){
            this.$dialog.notify({mes: '追加内容过短', icon: 'error', timeout: 1200});
            return;
          }else{
            _url =_url +'&askCtx='+this.answerText;
          }
        }else{
          _url =_url +'&images='+this.images;
        }
        this.$dialog.loading.open('加载中');
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.BTN_PAGE_INTERACTIONDETAIL_SAVE_APPLY});
        that.$http.get(this.apiAnswerAddUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              that.answerAsList.push(data.result);
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_INTERACTIONDETAIL_SAVE_APPLY_OK});
              // this.$router.back();
              // this.answerAsList = data.result;
            }else if(data.msg=='unauth'){
              // that.getLoginData(2);
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_INTERACTIONDETAIL_SAVE_APPLY_FAIL});
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
      //获取本地图片信息
      readFile(type,n,str){ 
        let that = this;
        var formDOM = document.getElementById('input-file');
        //将form的DOM对象当作FormData的构造函数
        var formData = new FormData(formDOM);
        console.log(formData,'formData');
        var req = new XMLHttpRequest();
        this.$dialog.loading.open('上传中...');
        this.tips4 = true;
        req.open("POST", window.interactionBaseUrl+'/upload/uploadImg2');
        //请求完成回调
        req.onload = function(){
            if(this.status === 200){
              var _data = JSON.parse(this.response);
              that.images = _data.result[0];
              that.saveSubmit('iamges');
            }
            that.$dialog.loading.close();
            that.tips4 = false;
         }
         //将form数据发送出去
        req.send(formData);
　　　　//避免内存泄漏
　　　　req = null; 
      },
       
       
      //获取追问列表
      getAnswerAskData(Pfile){
        var that = this;
        var _id= this.$route.query.answerId;
        var _url = this.interactionOpt.token + '&memberId=' + this.interactionOpt.memberId +'&answerId='+_id+'&orderBy=asc';
        this.opts = _url;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.apiAnswerAskUrl+this.opts+'&pageNumber='+this.pageNumber)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.answerAsList = data.result;
              this.getCurUser();
            }else if(data.msg=='unauth'){
              that.getLoginData(1);
              console.log(data.msg,'data-msg');
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }else{
              console.log(data.msg,'data-msg');
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
      }
    }
    
  }
</script>