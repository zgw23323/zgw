<template>
  <section class="ck-doc maintainParts">
    <article class="part_content">
      <p class="title_mess T3C5 i_border_tommon">建议保养以下配件</p>
      <ul>
        <li v-for="part,index in maintainPartsList" @click="partBtn(index)">
          <yd-cell-between-item :type="'y_gou'" :dataOpt="part"></yd-cell-between-item>
        </li>
      </ul>
    </article>
    <div class="interaction_btn1 T1C7" @click="saveSubmit">完成</div>
  </section>

</template>

<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore,setStore,getStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  import {ydFunLib} from '../../utils/ydFunLib';
  export default {
    data() {
      return {
        title: '选择保养配件',
        tips4:false,
        apiAnswerAddUrl: window.interactionBaseUrl+'/answer/add?token=',//新增答案
        apiAnswerUpdateUrl: window.interactionBaseUrl+'/answer/update?token=',//修改答案
        apiMaintainPartsUrl: window.interactionBaseUrl+'/answer/partList?token=',//获取配件列表
        answerData: {},
        maintainPartsList:[]
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由0.005rem助初始化）
       
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
       this.getMaintainPartData();
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
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_MAINTAINPARTS});
        var _questionDetailData = getSessionStore('questionDetailData',true);
        if(typeof(_questionDetailData) != 'boolean'){
          this.answerData = _questionDetailData;
          if(!_questionDetailData.maintance){
            this.answerData.maintance = [];
          }
        }
        var _interactionData = getStore('interactionData',true);
        if(typeof(_interactionData) != 'boolean'){
          this.answerData.token = _interactionData.token;
          this.answerData.memberId = _interactionData.memberId;
          console.log(this.answerData);
        }
      },
      partBtn(pIndex) {
        this.maintainPartsList[pIndex].isCur = !this.maintainPartsList[pIndex].isCur;
      },
      //完成
      saveSubmit() {
        let that = this;
        var partIdArr = [];
        var partNameArr = [];
        for(var i=0;i<this.maintainPartsList.length;i++){
          var item = this.maintainPartsList[i];
          if(item.isCur){
            partIdArr.push(item.id);
            partNameArr.push(item.name);
          }
        }
        var opt = {
          images: this.answerData.images.join(','),
          isExistMaintance: partIdArr.length>0? '1' : '0',
          partId: partIdArr.length>0 ? partIdArr.join(',') :'',
          partName: partNameArr.length>0 ? partNameArr.join(',') :''
        };
        var _url = this.answerData.token+'&memberId='+this.answerData.memberId+'&questionId='+this.answerData.questionId+'&answerMemberId='+this.answerData.memberId+'&answerCtx='+this.answerData.answerCtx+'&isExistMaintance='+opt.isExistMaintance;
        if(opt.images){
          _url = _url+'&images='+ opt.images;
        }
        if(partIdArr.length>0){
          _url = _url+'&partId='+opt.partId+'&partName='+opt.partName;
        }
        console.log('answerQuestion=>save',_url);
        this.$dialog.loading.open('保存中...');
        if(this.$route.query.type==1){//新增
          _url = this.apiAnswerAddUrl+_url;
        }else{//修改
          _url = this.apiAnswerUpdateUrl+_url+'&id='+this.answerData.id;
        }
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.BTN_PAGE_MAINTAINPARTS_SAVE_APPLY});
        that.$http.get(_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              that.$dialog.notify({mes: '保存成功', icon: 'success', timeout: 2000,callback: () => {
                that.$router.push({ path: 'questionDetail', query: { 'pid': this.answerData.questionId }});
              }});
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_MAINTAINPARTS_SAVE_APPLY_OK});
            }else {
              that.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_MAINTAINPARTS_SAVE_APPLY_FAIL});
              console.log('data.msg=>',data.msg);
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
       
      //获取配件列表
      getMaintainPartData(){
        var that = this;
        this.$dialog.loading.open('加载中');
        console.log('this.answerData->',this.answerData);
        that.$http.get(this.apiMaintainPartsUrl+this.answerData.token+'&memberId='+this.answerData.memberId)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              var _maintain = this.answerData.maintance;
              for(var i=0;i<data.result.length;i++){
                var item = data.result[i];
                if(_maintain.length>0){
                  var len = this.answerData.maintance.filter(function(data){
                    return data.partId == item.id;
                  });
                  data.result[i].isCur = len.length>0?true:false;
                }else{
                  data.result[i].isCur = false;
                }
              }
              this.maintainPartsList = data.result;
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