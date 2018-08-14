<template>
  <section class="questionArticleDetail raidersArticleDetail">
    <article class="part_content">
      <p class="T1C7 p_title" v-if="raidersArticleDetail.firstTitle !=''">{{raidersArticleDetail.firstTitle}}</p>
      <p class="T1C7 p_title" v-else="raidersArticleDetail.secondTitle !=''">{{raidersArticleDetail.secondTitle}}</p>
      <ul class="article_Info">
        <li v-for="question in raidersArticleDetail.questions">
          <p class="T1C3">{{question.title}}</p>
          <div class="T3C4" v-html="question.content"></div>
        </li>
        <li>
          <p class="T1C3">以下专家解决过相关问题：</p>
          <div class="i_flex master_list" v-if="master" v-for="master in masterList">
            <div class="it_1"><img :src="master.logo" /></div>
            <div>
              <div class="T3C3">{{master.name}}</div>
              <div class="T6C5"><span style="margin-right: .2rem;">回答数：<span class="T6C4">{{master.answerNum}}</span></span><span>被采纳：<span class="T6C4">{{master.adoptNum}}</span></span></div>
            </div>
          </div>
        </li>
      </ul>
    </article>
    <div :class="{'fixed_bottom T5C4':!isHas,'fixed_bottom T5C2':isHas}" @click="hasBtn"><i :class="{'i_has i_gou_no':!isHas,'i_has i_gou':isHas}">&nbsp;</i><span>我也遇到过</span></div>
  </section>

</template>

<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,getStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '攻略文章详情',
        interactionOpt:{},
        apiRaidersArticleDetailUrl: window.interactionBaseUrl+'/raidersArticle/detail?token=',//获取攻略文章详情
        apiMasterInfoUrl: window.interactionBaseUrl+'/master/masterInfo?token=',//获取师傅信息
        apiReportUrl: window.interactionBaseUrl + '/guess/report?token=',//上报问题
        apiCancelUrl: window.interactionBaseUrl + '/guess/cancel?token=',//取消上报
        masterList:[],
        opts: {
          token: '',
          articleId: '',
        },
        isHas:false,
        raidersArticleDetail:{}
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由0.005rem助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchRaidersArticleDetail();
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
        var _interactionData = getStore('interactionData',true);
        if(typeof(_interactionData) != 'boolean'){
          this.interactionOpt = _interactionData;
        }
        this.opts.token = this.interactionOpt.token;
        this.opts.articleId = this.$route.query.articleId;
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_ARIDERSARTICLE});
      },
      //我也遇到过类似问题点击
      hasBtn() {
        var _isHas = this.isHas ? false : true;
        if(_isHas){
          this.fetchReport();
        }else{
          this.fetchCancel();
        }
      },
      //上报问题
      fetchReport() {
        var that = this;
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.BTN_PAGE_RAIDERSARTICLEDETAIL_REPORT_APPLY});
        that.$http.get(this.apiReportUrl+this.interactionOpt.token+'&articleId='+this.opts.articleId)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.isHas = true;
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_RAIDERSARTICLEDETAIL_REPORT_APPLY_OK});
            }else{
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_RAIDERSARTICLEDETAIL_REPORT_APPLY_FAIL});
            }
          },(response) => {
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //取消上报
      fetchCancel() {
        var that = this;
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.BTN_PAGE_RAIDERSARTICLEDETAIL_REPORT_CANCEL});
        that.$http.get(this.apiCancelUrl+this.interactionOpt.token+'&articleId='+this.opts.articleId)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.isHas = false;
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_RAIDERSARTICLEDETAIL_REPORT_CALCEL_OK});
            }else{
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_RAIDERSARTICLEDETAIL_REPORT_CANCEL_FAIL});
            }
          },(response) => {
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //获取师傅信息
      fecthMasterData(){
        var that = this;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.apiMasterInfoUrl+this.interactionOpt.token+'&masterIdList='+this.raidersArticleDetail.masters)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.masterList = data.result;
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
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
      //获取攻略文章详情
      fetchRaidersArticleDetail(){
        var that = this;
        this.$dialog.loading.open('加载中');
        console.log('fetchRaidersArticleDetail');
        that.$http.get(this.apiRaidersArticleDetailUrl+this.interactionOpt.token+'&articleId='+this.opts.articleId)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.raidersArticleDetail = data.result;
              if(data.result.masters != null){
                that.fecthMasterData();
              }
            }else {
              console.log('data-msg->',data.msg);
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
      }
    }
    
  }
</script>