<template>
  <section class="ck-doc questionHot masterWorker T3C3">
    <div class="fixed_top">
      <div class="ck-search home-search-bar i_flex_align">
          <div class="i_eacrch">
            <i class="iconfont">&#xe602;</i>
            <input type="text" v-model="searchKey" placeholder="搜索您想知道的问题" />
          </div>
          <div class="T3C7" @click="cancelBack">取消</div>
      </div>
      
    </div>
    <div v-if="questionList && questionList.length==0">
      <!-- 输入框自动匹配搜索 -->
      <ul class="question_se_list" v-if="questionSe && questionSe.length>0">
        <li v-for="item,index in questionSe" :class="{'no_bor':index==questionSe.length-1}" @click="searchQuestion(item.tags,item.id)">{{item.tags}}</li>
      </ul>
      <article class="questionHot_list" v-if="questionSe && questionSe.length==0">
        <ul class="li_se">
          <li v-for="item,index in liQuestionSe" :class="{'i_flex_align no_bor':index==liQuestionSe.length-1&& liQuestionSe.length<=3,'i_flex_align':index<=liQuestionSe.length-1}">
            <p @click="searchQuestion(item.content,item.id)">{{item.content}}</p>
            <div @click="deleteTags(index)"><i class="tag_del">&nbsp;</i></div>
          </li>
          <li class="T3C2 no_bor text_cen" v-if="liQuestionSe&&liQuestionSe.length>3" @click="seBtn" ><span v-if="isShowMore">清除历史</span><span v-else>更多</span></li>
        </ul>
        <div v-if="questionHotist && questionHotist.length>0">
          <div class="i_flex_align hot_head" >
            <div><i class="hot_logo">&nbsp;</i><span>热门搜索</span></div>
            <div class="T5C2" @click="fetchQuestionHot"><i class="hot_ch">&nbsp;</i><span>换一批</span></div>
          </div>
          <ul class="li_hot i_flex_align_center">
            <li v-for="item,index in questionHotist"  @click="searchQuestion(item.content,item.id)" :class="{'no_bor':index==questionHotist.length-1 || index==questionHotist.length-2}" ><span>{{item.content}}</span></li>
          </ul>
        </div>
      </article>
    </div>
    <div v-if="questionList && questionList.length>0" style="padding-top: .88rem;">
      <ul class="interaction_list">
        <li class="li1" v-for="question in questionList" @click="detailsGo(question.id)">
          <div class="item_head i_flex_align">
            <div>
              <img :src="question.questionerImg" v-if="question.questionerImg" class="img1" />
              <img src="http://image.xiaobaicar.com/default/head/genius.png" v-else class="img1" />
              <span class="i_phone" v-if="question.questionerName">{{question.questionerName}}</span>
              <span class="i_phone" v-else>{{interactionOpt.mobile.substring(0,3)}}****{{interactionOpt.mobile.substring(7,11)}}</span>
              <i class="u_car_logo" :style="{backgroundImage: 'url('+question.carLogo + ')'}">&nbsp;</i>
              <!-- <span class="g_km">{{question.distance}}</span> -->
            </div>
            <div class="isEssence" v-if="question.isEssence==1">精华</div>
          </div>
          <div class="item_con">
            <div class="b_1"><span class="q_mess">{{question.questionCtx}}<span class="q_people" v-for="item in question.inviterName">{{item}}</span></span></div>
            <div>
              <i class="fault_logo">&nbsp;</i>
              <span class="fault_mess" v-for="item in question.tags">{{item}}</span>
            </div>
          </div>
          <div class="item_botton i_flex_between">
            <div class="q_time">{{question.createTime.split(' ')[0]}}</div> 
            <div>
              <span class="s_mess" v-if="question.isAdoptedAnswer==1"><i class="s_secess">&nbsp;</i>已解决</span>
              <span class="l_count" v-else-if="question.isClosed == 0"><i class="s_over">&nbsp;</i>未解决</span>
              <span class="l_count" v-else><i class="s_over">&nbsp;</i>已结束</span>
              <span class="l_count"><i class="count_logo">&nbsp;</i>{{question.answerNum}}</span>
            </div>
          </div> 
        </li>
      </ul>
      <div class="fixed_bottom b_top_10">
        <div class="quest_btn b_color_2 T3C7" @click=""><router-link to="/question"><i class="quest_icon_3">&nbsp;</i>提问</router-link></div>
      </div>
    </div>
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
        title: '热门搜索',
        searchKey:'',
        interactionOpt:{},
        isShowMore: false,
        pageNumber:0,
        searchFlag:true,
        _url:'',
        isMax:false,
        apiQuestionHotUrl: window.interactionBaseUrl+'/questionHot/list?token=',//获取热门搜索列表
        apiQuestionHotSeUrl: window.interactionBaseUrl+'/question/se?token=',//热门标签搜索
        apiQuestiUrl: window.interactionBaseUrl+'/question/search?token=',//搜索问题列表
        questionHotist:[],
        isHas:false,
        liQuestionSe:[],
        questionList: [],//问题列表
        questionSe:[]
      }
    },
    watch: {
      '$route.path': 'markInit',//监听当前路由助初始化）
      'searchKey'(){
        let _searchKey = ydFunLib.trim(this.searchKey,'g');
        if(this.searchFlag){
          if(_searchKey != "") {
            if(this.searchFlag) {
              this.searchFlag = false;
              this.fetchSearchTags(_searchKey);
              setTimeout(()=>{
                this.searchFlag = true;
              },3000);
            }
          }else {
            this.questionSe = [];
          }
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      var _interactionData = getStore('interactionData',true);
      if(typeof(_interactionData) != 'boolean'){
        this.interactionOpt = _interactionData;
      }

      var _questionHot = getStore('xiao_bai_question_hot',true);
      if(typeof(_questionHot) != 'boolean'){
        this.liQuestionSe = _questionHot;
      }
      this.fetchQuestionHot();
      window.addEventListener('scroll', this.menu);
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_QUESTIONHOT});
      },
      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
            this.loadList();
        }
      },
      loadList() {
        //分页加载异步请求数据
        let that = this;
        that.pageNumber++;
        if(this.isMax) {
          /* 所有数据加载完毕 */
          that.isLoading = false;
          return;
        }
        this._url = this._url+'&pageNumber='+this.pageNumber; 
        that.$http.get(that.apiQuestiUrl+this._url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
              for (var i=0;i<data.result.length;i++) {
                var item = data.result[i];
                if(item.tags){
                  data.result[i].tags = item.tags.split(',');
                }
                if(item.questionerImg){
                  data.result[i].questionerImg = item.questionerImg.split('http:').length>1?item.questionerImg:'https://api.xiaobaicar.com'+item.questionerImg;
                }
                if(item.inviterName){
                  data.result[i].inviterName = item.inviterName.split(',');
                }
                that.questionList.push(data.result[i]);
              }
              that.isMax = data.result.length>0? false:true;
            }else {
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      //后退
      cancelBack() {
        this.$router.back();
      },
      //搜索标签问题列表
      searchQuestion(str,pId){
        this.searchKey = str;
        this.searchFlag = false;
        this.fetchQuestionData(str,pId);
      },
      //我也遇到过类似问题点击
      hasBtn() {
        this.isHas = !this.isHas;
      },
      detailsGo(id) {
        //跳去商品详情
        this.$router.push({ path: 'questionDetail', query: { 'pid': id } });
      },
      //历史搜索点击
      seBtn(){
        if(this.isShowMore){
          this.liQuestionSe = [];
        }else{
          this.isShowMore = true;
        }
      },
      //删除单项历史记录标签
      deleteTags(n) {
        this.liQuestionSe.splice(n,1);
      },
      //获取热门标签列表
      fetchQuestionHot(){
        var that = this;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.apiQuestionHotUrl+this.interactionOpt.token+'&pageSize='+6)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.questionHotist = data.result;
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
            
          },(response) => {
            that.$dialog.loading.close();
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //获取搜索标签列表
      fetchSearchTags(str){
        var that = this;
        that.$http.get(this.apiQuestionHotSeUrl+this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId+'&text='+str)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.questionSe = data.result;
            }
          },(response) => {
            that.$dialog.loading.close();
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //搜索问题列表
      fetchQuestionData(str,pId) {
        var that = this;
        var _opt = {
          token: this.interactionOpt.token,
          memberId: this.interactionOpt.memberId,
          pageNumber: 0,
          id:pId,
          content: str
        };
        var len = this.liQuestionSe.filter(function(data){
          return data.id != pId;
        });
        if(len.length==0){
          that.liQuestionSe.push(_opt);
        }
        var _url = this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId+'&text='+str+'&pageNumber=0';
        that._url = _url;
        setStore('xiao_bai_question_hot',that.liQuestionSe);
        that.$dialog.loading.open('加载中');
        that.$http.get(this.apiQuestiUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              let questionArr = [];
              for (var i=0;i<data.result.length;i++) {
                var item = data.result[i];
                if(item.tags){
                  data.result[i].tags = item.tags.split(',');
                }
                if(item.questionerImg){
                  data.result[i].questionerImg = item.questionerImg.split('http:').length>1?item.questionerImg:'https://api.xiaobaicar.com'+item.questionerImg;
                }
                if(item.inviterName){
                  data.result[i].inviterName = item.inviterName.split(',');
                }
                questionArr.push(data.result[i]);
              }
              that.questionList = questionArr;
              that.isMax = data.result.length>0? false:true;
            }else {
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = false;
            },300);
            
          },(response) => {
            that.$dialog.loading.close();
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      }
    }
    
  }
</script>