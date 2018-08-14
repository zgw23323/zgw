 <template>

  <section class="ck-doc interactionHome">

    <div v-if="!tips1">
      <div class="ck-search home-search-bar">
        <router-link to="questionHot">
          <div class="i_eacrch">
            <i class="iconfont">&#xe602;</i>
            <span>搜索你想知道的问题</span>
          </div>
        </router-link>
      </div>
      <div class="home-banner">
        <yd-slider v-if="bannerList && bannerList.length>0" autoplay="3000" :ready="start1">
          <yd-slider-item :key="banner.id" v-for="banner,index in bannerList">
            <div @click="goTo('questionArticleDetail',banner.articleId)" class="bannerimg" :style="{backgroundImage: 'url(' + banner.pic + ')'}">
              <p class="T1C7">{{banner.firstTile}}</p>
            </div>
          </yd-slider-item>
        </yd-slider>
        <router-link :to="{path:'questionArticle'}" class="tip_banner">
          <yd-cell-between-item :title="'发现更多'" :type="'right'" :dataOpt="{}"></yd-cell-between-item>
        </router-link>
      </div>
      <ul class="nav_strategy" v-if="raidersList && raidersList.length>0">
        <li v-for="raiders,index in raidersList" @click="goTo('raidersArticle',raiders)">
            <i class="icon_1" :style="{backgroundImage: 'url(' + raiders.imageUrl + ')'}">&nbsp;</i>
            <span>{{raiders.name}}</span>
        </li>
        <li  @click="goTo('raidersArticle')">
            <i class="icon_2 T1C7">···</i>
            <span>更多</span>
        </li>
      </ul>
      <ul class="interaction_list">
        <li class="li1" v-for="question in questionList" @click="detailsGo(question.id)">
          <div class="item_head i_flex_align">
            <div>
              <img :src="question.questionerImg" v-if="question.questionerImg" class="img1"/>
              <img src="http://image.xiaobaicar.com/default/head/genius.png" v-else class="img1" />
              <span class="i_phone" v-if="question.questionerName">{{question.questionerName}}</span>
              <span class="i_phone" v-else>{{interactionOpt.mobile.substring(0,3)}}****{{interactionOpt.mobile.substring(7,11)}}</span>
              <i class="u_car_logo" v-if="question.carLogo" :style="{backgroundImage: 'url('+question.carLogo + ')'}">&nbsp;</i>
              <!-- <span class="g_km">{{question.distance}}</span> -->
            </div>
            <div class="isEssence" v-if="question.isEssence==1">精华</div>
          </div>
          <div class="item_con">
            <div class="b_1"><span class="q_mess">{{question.questionCtx}}<span class="q_people" v-for="item in question.inviterName">{{item}}</span></span></div>
            <div v-if="question.tags">
              <i class="fault_logo">&nbsp;</i>
              <span class="fault_mess" v-for="item in question.tags">{{item}}</span>
            </div>
          </div>
          <div class="item_botton i_flex_between">
            <div class="q_time">{{question.createTime.split(' ')[0]}}</div>
            <div>
              <span class="s_mess" v-if="question.isAdoptedAnswer==1"><i class="s_secess">&nbsp;</i>已解决</span>
             <!--  <span class="l_count" v-else-if="question.isClosed == 0"><i class="s_over">&nbsp;</i>未解决</span> -->
              <span class="l_count" v-if="question.isAdoptedAnswer==0&&(question.isClosed==1||question.isClosed==-1)"><i class="s_over">&nbsp;</i>已结束</span>
              <span class="l_count" v-if="question.answerNum"><i class="count_logo">&nbsp;</i>{{question.answerNum}}</span>
            </div>
          </div>
        </li>
      </ul>
    </div>
    <div class="tipsPage" v-if="tips1">
      <div class="tipsBox">
        <div class="tipStatus" data-type="busy"></div>
        <p>亲，网络出问题啦~~</p>
        <div class="btns" @click="reLoad">重新加载</div>
      </div>
    </div>

    <div class="question_fixed_btn"><router-link to="question" class="q_logo">&nbsp;</router-link></div>
  </section>
</template>
<script type="text/babel">
  document.title = '友道会商城';
  import {setShopsId,setSessionStore,removeSessionStore,getSessionStore,setStore,getStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        tips1: false,
        tips2:false,
        isSingle: false,
        start1: true,
        title: '问答',
        imgBaseUrl: window.imgBaseUrl,
        interactionOpt:{
          memberId:'',
          token:'',
          mobile:'123456789009',
          memberCar:[]
        },
        raidersList:[],
        isMax:false,
        mainUrl:window.loginRegisterUrl+'/loginApi/login/loginYd?uid=',
        registerUrl:window.loginRegisterUrl+'/loginApi/register/registerYd?uid=',
        apiUrl: window.interactionBaseUrl+'/question/list?token=',//获取问题列表
        apiRaidersUrl: window.interactionBaseUrl+'/raiders/index_v2?token=',//获取首页攻略列表
        apiQuestionArticle: window.interactionBaseUrl + '/questionArticle/recommends?token=',//获取首页推荐文章列表
        opts: {pageNumber: 0,pageSize:8},
        bannerList: [],
        optAction: {},//活动信息对象
        questionList: []
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
          this.fetchHomeData();
          this.fetchRaiderData();
          this.fetchQuestionArticleData();
      }else{
        this.getLoginData();
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_INTERACTIONHOME});
      },
      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
            this.loadList();
        }
      },
      detailsGo(id) {
        //跳去商品详情
        this.$router.push({ path: 'questionDetail', query: { 'pid': id } });
      },
      //跳去攻略列表
      goTo(str,opt){
        if(str == 'raidersArticle'){
          if(opt){
            setSessionStore('raiders_data',opt);
          }else{
            removeSessionStore('raiders_data');
          }
          this.$router.push({ path: str});
        }else if(str =='questionArticleDetail'){
          this.$router.push({ 
            path: str,
            query: {articleId: opt}
          });
        }
      },
      loadList() {
        //分页加载异步请求数据
        let that = this;
        that.opts.pageNumber++;
        if(this.isMax) {
          /* 所有数据加载完毕 */
          that.isLoading = false;
          // that.isDone = true;
          // that.$yduiBus.$emit('ydui.infinitescroll.loadedDone');
          return;
        }
        that.$http.get(that.apiUrl+this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId+'&pageNumber='+that.opts.pageNumber+'&pageSize=8')
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
              that.isMax = data.result.length>0? false:true;
              for (var i=0;i<data.result.length;i++) {
                var item = data.result[i];
                if(item.tags){
                  data.result[i].tags = item.tags.split(',');
                }
                if(item.inviterName){
                  data.result[i].inviterName = item.inviterName.split(',');
                }
                if(item.carLogo){
                  data.result[i].carLogo = item.carLogo.split('http:').length>1?item.carLogo:'';
                }
                that.questionList.push(data.result[i]);
              }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      //登录
      getLoginData() {
        var that = this;
        ydFunLib.interactionLogin(function(data){
            that.interactionOpt.memberId = data.memberId;
            that.interactionOpt.token = data.token;
            setStore('interactionData',that.interactionOpt);
            that.fetchHomeData();
            that.fetchRaiderData();
            that.fetchQuestionArticleData();
            that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_INTERACTIONHOME_LOGIN_OK});
        })
      },
      //获取首页推荐文章列表
      fetchQuestionArticleData() {
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.get(that.apiQuestionArticle+this.interactionOpt.token)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "s_ok") {
              that.bannerList = data.result;
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              that.getLoginData();
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            // setTimeout(()=>{
            //   that.$dialog.loading.close();
            // },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      //获取首页攻略列表 
      fetchRaiderData() {
        let that = this;
        console.log('获取首页攻略->');
        that.$dialog.loading.open('加载数据中...');
        that.$http.get(that.apiRaidersUrl+this.interactionOpt.token)
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "s_ok") {
              that.raidersList = data.result;
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              that.getLoginData();
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            // setTimeout(()=>{
            //   that.$dialog.loading.close();
            // },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      goDetails(id,serveCode,prodSource) {
          //跳去详情页
          if(serveCode == '' || prodSource !='4'){
            this.$router.push({
              path: '/details',
              query: {
                pid: id
              }
            });
          }else{
            var str = serveCode.split('_')[0];
            this.$router.push({
              path: '/serviceDetail',
              query: {
                'pid': id,
                'serveCode':str,
                'psource':1
              }
            });
          }
      },
      fetchHomeData: function() {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.get(that.apiUrl+this.interactionOpt.token+'&pageNumber=0'+'&memberId='+this.interactionOpt.memberId+'&pageSize=8')
          .then((response) => {
            const data = response.data;
            that.tips1 = false;
            if(data.code == "s_ok") {
              let questionArr = [];
              for (var i=0;i<data.result.length;i++) {
                var item = data.result[i];
                if(item.tags){
                  data.result[i].tags = item.tags.split(',');
                }
                if(item.inviterName){
                  data.result[i].inviterName = item.inviterName.split(',');
                }
                if(item.carLogo){
                	data.result[i].carLogo = item.carLogo.split('http:').length>1?item.carLogo:'';
                }
                questionArr.push(data.result[i]);
              }
              
              that.questionList = questionArr;
              that.isMax = data.result.length>0? false:true;
            }else if(data.error_code == '110'&&data.code=="s_fail"){
              that.getLoginData();
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

<style lang="less">
  @import "../../styles/common/interaction.less";
</style>
