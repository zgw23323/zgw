<template>
  <section class="ck-doc maintainParts tags b_color_8">
    <article class="part_content">
      <!-- <p class="title_mess T3C5 i_border_tommon">建议保养以下配件</p> -->
      <ul class="b_color_7">
        <li v-for="tag,index in tagsList" @click="tagBtn(index)">
          <yd-cell-between-item :type="'y_gou'" :dataOpt="tag"></yd-cell-between-item>
        </li>
      </ul>
    </article>
    <div class="interaction_btn2 T3C2" @click="saveSubmit"><span class="tagCount" v-if="tagCount>0">{{tagCount}}</span>完成</div>
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
        title: '选择标签',
        tips4:false,
        tagCount:0,
        apiTagsUrl: window.interactionBaseUrl+'/tag/hot?token=',//获取配件列表
        interactionOpt:{
          memberId:'',
          token:'',
          mobile:'123456789009'
        },
        tagsList:[],
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
      var _questionOptData = getSessionStore('questionSubmitData',true);
      if(typeof(_questionOptData) != 'boolean'){
        this.applyOpt = _questionOptData;
      }
      this.getTagData();
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_TAGS});
      },
      tagBtn(pIndex) {
        if(this.tagCount==3 && !this.tagsList[pIndex].isCur){
          this.$dialog.notify({mes: '最多只能选择3个标签！', icon: 'error', timeout: 1200});
          return false;
        }
        this.tagsList[pIndex].isCur = !this.tagsList[pIndex].isCur;
        var len = this.tagsList.filter(function (data) {
          return data.isCur;
        });
        this.tagCount = len.length;
      },
      //完成
      saveSubmit() {
        if(this.tagCount==0){
          this.$dialog.notify({mes: '请选择问题标签！', icon: 'error', timeout: 1200});
          return false;
        }
        var tagArr = [];
        for(var i=0;i<this.tagsList.length;i++){
          if(this.tagsList[i].isCur){
            tagArr.push(this.tagsList[i].name);
          }
        }
        this.applyOpt.tags = tagArr.join(',');
        setSessionStore('questionSubmitData',this.applyOpt);
        this.$router.back();
      },
       
      //获取热门标签
      getTagData(Pfile){
        var that = this;
        var opt = {
          memberId : this.interactionOpt.memberId,
          token: this.interactionOpt.token
        };
        this.$dialog.loading.open('加载中');
        that.$http.get(this.apiTagsUrl+this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              var _tagArr = this.applyOpt.tags?this.applyOpt.tags.split(','):[];
              for(var i=0;i<data.result.length;i++){
                var item = data.result[i];
                if(_tagArr.length>0){
                  var len = _tagArr.filter(function(data){
                    return data == item.tagName
                  });
                  data.result[i].isCur = len.length>0? true:false;
                }else{
                  data.result[i].isCur = false;
                }
                data.result[i].name = data.result[i].tagName;
              }
              this.tagsList = data.result;
              this.tagCount = _tagArr.length;
            }else {
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