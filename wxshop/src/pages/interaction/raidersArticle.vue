<template>
  <section class="ck-doc raidersArticle T5C3">
    <article class="raiders_nav fixed_top">
      <div :class="{'nav_btn T5C2 i_border_tommon i_flex_align_center':tips4,'T5C4 nav_btn i_border_tommon i_flex_align_center':!tips4}" @click="nvgBtn">
        <span>{{raidersMess1}}<span style="margin-left: .1rem;">{{raidersMess2}}</span></span>
        <i :class="{'arrow_dowm_cur':tips4,'arrow_dowm_no':!tips4}">&nbsp;</i>
      </div>
      <div class="raiders_nag_list i_flex" v-if="tips4">
        <ul class="left">
          <li v-for="item in raidersTree" :class="{'cur': curBtnId == item.id}" @click="raidersBtn(item)">{{item.name}}</li>
        </ul>
        <ul class="right">
          <li class="i_flex_align" v-for="item in raidersTreeChren.childrens" @click="raidersTwoBtn(item)">
            <div>{{item.name}}</div>
            <div v-if="curTreeId == item.id"><i class="y_gou">&nbsp;</i></div>
          </li>
        </ul>
      </div>
    </article>
    <div class="all_rgba" v-if="tips4">&nbsp;</div>
      <!-- 攻略列表 -->
    <ul class="b_color_7 raiders_list">
      <li v-for="article,index in articleList" @click="articleBtn(article)" class="i_flex_align">
        <div>{{article.title}}</div>
        <div><i class="i_right_logo">&nbsp;</i></div>
      </li>
      <li class="text_cen"><router-link to="questionHot" class="T5C2">关于{{raidersMess1}}{{raidersMess2}}的问题</router-link></li>
    </ul>
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
        title: '用车攻略',
        tips4:false,
        tagCount:0,
        raidersMess1:'全部',//一级标签选中名字
        raidersMess2:'',//二级标签选中名字
        curBtnId:'',
        curTreeId:'-1',
        apiQuestionArticleUrl: window.interactionBaseUrl+'/raidersArticle/all_v2?token=',//获取攻略文章列表
        apiRaidersTreeUrl: window.interactionBaseUrl+'/raiders/tree?token=',//获取攻略树列表
        interactionOpt:{
          memberId:'',
          token:'',
          mobile:'123456789009'
        },
        opts: {
          token:'',
          carId: '',
          pageNumber:0,
          raidersPartId:'',//攻略部位ID
          raidersPheId:''//攻略现象ID
        },
        _url:'',
        isMax: false,//当前是否是最大页数
        raidersTree: [],//攻略树列表
        raidersTreeChren: {},//攻略树二级列表
        articleList:[]
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由0.005rem助初始化）
       
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      var raiders_opt = getSessionStore('raiders_data',true);
       if(typeof(raiders_opt) != 'boolean'){
        this.opts.raidersPartId = raiders_opt.id;
        this.raidersMess1 = raiders_opt.name;
      }
      var _interactionData = getStore('interactionData',true);
      if(typeof(_interactionData) != 'boolean'){
        this.interactionOpt = _interactionData;
      }
      this.getQuestionArticleData();
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_ARIDERSARTICLE});
      },

      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
            this.loadList();
        }
      },
      //点击攻略分类
      nvgBtn() {
        if(this.raidersTree.length==0){
          this.fetchRaidersTreeData();
        }
        this.tips4 = !this.tips4;
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
        this._url = this._url+'&pageNumber='+this.opts.pageNumber; 
        that.$http.get(that.apiQuestionArticleUrl+this._url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
                for (let list of data.result) {
                  that.articleList.push(list);
                }
                that.isMax = data.result.length>0? false:true;
            }else {
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      //攻略文章点击
      articleBtn(opt) {
        this.$router.push({ 
          path: '/raidersArticleDetail',
          query: {articleId: opt.articleId}
        });
      },
      //攻略一级标签点击
      raidersBtn(opt) {
        console.log('raiders=>',opt);
        this.curBtnId = opt.id;
        this.raidersTreeChren = opt;
        this.opts.raidersPartId = this.curBtnId;
        this.opts.pageNumber = 0;
      },
      //攻略树二级标签点击
      raidersTwoBtn(opt){
        this.curTreeId = opt.id;
        this.opts.raidersPheId = opt.id;
        this.opts.pageNumber = 0;
        this.raidersMess1 = this.raidersTreeChren.name;
        this.raidersMess2 = opt.name;
        this.getQuestionArticleData();
      },
      //获取攻略树列表
      fetchRaidersTreeData(){
        let that = this;
        that.$http.get(that.apiRaidersTreeUrl+this.interactionOpt.token)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
              that.raidersTree = data.result;
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      //获取攻略文章列表
      getQuestionArticleData(bol){
        var that = this;
        this.$dialog.loading.open('加载中');
        var _url = this.interactionOpt.token+'&pageNumber=0';
        if(this.opts.raidersPheId !=''){
          _url = _url+'&raidersPheId='+this.opts.raidersPheId;
        }
        if(this.opts.raidersPartId !=''){
          _url = _url+'&raidersPartId='+this.opts.raidersPartId;
        }
        this._url = _url;
        that.$http.get(this.apiQuestionArticleUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              if(data.result && data.result.length>0){
                this.articleList = data.result;
                that.isMax = false;
              }else{
                that.isMax = true;
              }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.tips4 = bol;
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