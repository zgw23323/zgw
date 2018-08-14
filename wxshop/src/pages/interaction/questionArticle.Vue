<template>
  <section class="ck-doc questionArticle">
    <article class="part_content">
      <!-- <p class="title_mess T3C5 i_border_tommon">建议保养以下配件</p> -->
      <ul class="b_color_7">
        <li v-for="article,index in questionArticleList" @click="articleBtn(article)" :style="{backgroundImage: 'url(' + article.pic + ')'}">
        <p class="T1C7" v-if="article.firstTile">{{article.firstTile}}</p>
        <p class="T1C7" v-else="article.secondTitle">{{article.secondTitle}}</p>
        </li>
      </ul>
    </article>
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
        title: '发现更多',
        tips4:false,
        tagCount:0,
        apiQuestionArticleUrl: window.interactionBaseUrl+'/questionArticle/list?token=',//获取攻略文章列表
        interactionOpt:{
          memberId:'',
          token:'',
          mobile:'123456789009'
        },
        opts: {
          token:'',
          pageNumber: 0
        },
        isMax: false,//当前是否是最大页数
        questionArticleList:[]
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_QUESTIONARTICLE});
      },

      loadList() {
        //分页加载异步请求数据
        let that = this;
        that.opts.pageNumber++;
        if(isMax) {
          /* 所有数据加载完毕 */
          that.isLoading = false;
          // that.isDone = true;
          // that.$yduiBus.$emit('ydui.infinitescroll.loadedDone');
          return;
        }
        that.$http.get(that.apiQuestionArticleUrl+this.interactionOpt.token+'&pageNumber='+that.opts.pageNumber)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok") {
                for (let list of data.result) {
                  that.questionArticleList.push(list);
                }
                that.isMax = data.result.length>0? false:true;
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
      articleBtn(opt){
          this.$router.push({ 
            path: '/questionArticleDetail',
            query: {articleId: opt.articleId}
          });
      },
      //获取攻略文章列表
      getQuestionArticleData(Pfile){
        var that = this;
        this.opts.token = this.interactionOpt.token;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.apiQuestionArticleUrl+this.interactionOpt.token+'&pageNumber='+this.opts.pageNumber)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.questionArticleList = data.result.articles;
              that.isMax = data.result.length>0? false:true;
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
      }
    }
    
  }
</script>