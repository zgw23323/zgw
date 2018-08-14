<template>
  <section class="ck-doc questionArticleDetail">
    <article class="part_content">
      <p class="T1C7 p_title" v-if="questionArticleDetail.firstTitle !=''">{{questionArticleDetail.firstTitle}}</p>
      <p class="T1C7 p_title" v-else="questionArticleDetail.secondTitle !=''">{{questionArticleDetail.secondTitle}}</p>
      <ul class="article_Info">
        <li v-for="question in questionArticleDetail.questions">
          <p class="T1C3">{{question.title}}</p>
          <div class="T3C4" v-html="question.content"></div>
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
        interactionOpt:{},
        apiQuestionArticleDetailUrl: window.interactionBaseUrl+'/questionArticle/detail?token=',//获取推荐文章详情
        questionArticleDetail:{}
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
      this.fetchQuestionArticleDetail();
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_QUESTIONARTICLEDETAIL});
      },
      //获取推荐文章详情
      fetchQuestionArticleDetail(){
        var that = this;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.apiQuestionArticleDetailUrl+this.interactionOpt.token+'&articleId='+this.$route.query.articleId)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.questionArticleDetail = data.result;
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
      }
    }
    
  }
</script>