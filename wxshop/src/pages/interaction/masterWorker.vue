<template>
  <section class="ck-doc masterWorker">
    <div class="fixed_top">
      <div class="ck-search home-search-bar i_flex_align">
          <div class="i_eacrch">
            <i class="iconfont">&#xe602;</i>
            <input type="text" name="" v-model="searchMaster" placeholder="搜索师傅姓名" />
          </div>
          <!-- <div class="T3C7">完成</div> -->
      </div>
      <ul class="master_check_list i_flex_justify_center" v-if="masterCheck && masterCheck.length>0">
        <li v-for="master,index in masterCheck">
          <div class="m_head"><img :src="master.logo" /></div>
          <p class="T3C3">{{master.name}}</p>
          <div class="delete_ic" @click="deleteMaster(master,index)">&nbsp;</div>
        </li>
      </ul>
    </div>
    <article :class="{'masterWorker_content pa_1':masterCheck.length==0,'masterWorker_content pa_2':masterCheck.length>0}">
     
      <div class="search_list" v-if="searchMasterList && searchMasterList.length>0">
        <p class="title_mess T6C4 ">搜索结果</p>
        <ul class="b_color_7 master_list">
          <li :class="{'i_flex_align master_item i_border_tommon':index != searchMasterList.length-1,'i_flex_align master_item':index == searchMasterList.length-1}" v-for="master,index in searchMasterList" @click="masterBtn(master,index,'searchMasterList')">
             <div class="i_flex_align_center">
                <div class="m_head"><img :src="master.logo" /></div>
                <div>
                  <p class="T3C3">{{master.name}}</p>
                  <p class="T6C5 m_p1"><span>回答总数：<span class="T6C4">{{master.answerNum}}</span></span><span>被采纳：<span class="T6C4">{{master.adoptNum}}</span></span></p>
                </div>
             </div>
             <div><i :class="{'y_gou i_y_gou1':master.isCur,'y_gou i_y_gou2':!master.isCur}">&nbsp;</i></div>
          </li>
        </ul>
      </div>

      <div class="tui_list" v-if="masterList && masterList.length>0">
        <p class="title_mess T6C4 ">以下专家解决过相关问题</p>
        <ul class="b_color_7 master_list">
          <li :class="{'i_flex_align master_item i_border_tommon':index != masterList.length-1,'i_flex_align master_item':index == masterList.length-1}" v-for="master,index in masterList" @click="masterBtn(master,index,'masterList')">
             <div class="i_flex_align_center">
                <div class="m_head"><img :src="master.logo" /></div>
                <div>
                  <p class="T3C3">{{master.name}}</p>
                  <p class="T6C5 m_p1"><span>回答总数：<span class="T6C4">{{master.answerNum}}</span></span><span>被采纳：<span class="T6C4">{{master.adoptNum}}</span></span></p>
                </div>
             </div>
             <div><i :class="{'y_gou i_y_gou1':master.isCur,'y_gou i_y_gou2':!master.isCur}">&nbsp;</i></div>
          </li>
        </ul>
      </div>
    </article>
    <div class="interaction_btn2 T3C2" @click="saveSubmit">完成</div>
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
        title: '选择师傅',
        tips4:false,
        searchMaster:'',
        searchFlag: true,
        apiMasterUrl: window.interactionBaseUrl+'/question/commendMaster?token=',//获取被邀请师傅列表
        apiSearchMaster: window.interactionBaseUrl +'/question/seMaster?token=',//搜索师傅
        masterCheck:[],//选择的师傅列表
        searchMasterList:[],//搜索师傅列表
        masterList:[],
        applyOpt:{}
      }
    },
    watch: {
      '$route.path': 'markInit',//监听当前路由0.005rem助初始化）
      'searchMaster'(){
        let _searchKey = ydFunLib.trim(this.searchMaster,'g');
        console.log('this.searchMaster->',_searchKey);
        if(_searchKey != "") {
          if(this.searchFlag) {
            this.searchFlag = false;
            this.fetchSearchList(_searchKey);
            setTimeout(()=>{
              this.searchFlag = true;
            },3000);
          }
        }else {
          this.searchMasterList = [];
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

      var _questionOptData = getSessionStore('questionSubmitData',true);
      if(typeof(_questionOptData) != 'boolean'){
        this.applyOpt = _questionOptData;
        let that = this;
        var _masterArr = this.applyOpt.inviterList.split(',');
        for(var i=0;i<this.masterList.length;i++){
          if(_masterArr.length>0){
            var len = _masterArr.filter(function(data){
              return data == that.masterList[i].id
            });
            that.masterList[i].isCur = len.length>0? true:false;
          }else{
            that.masterList[i].isCur = false;
          }
          if(that.masterList[i].isCur){
            this.masterCheck.push(that.masterList[i]);
          }
        } 
      }
      this.fecthMasterList();
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
        // this.getTagData();
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.PAGE_MASTERWORKER});
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
        var _master = this.$route.query.masterId;
        if(_master){//被邀请提问的人ID
          this.applyOpt.inviter = _master;
        }
        if(this.masterCheck.length>0){
          var masterId = [];
          var masterName = [];
          for(var i=0;i<this.masterCheck.length;i++){
            var item = this.masterCheck[i];
            masterId.push(item.id);
            masterName.push('@'+item.name);
          }
        }
        this.applyOpt.inviterList = masterId&&masterId.length>0?masterId.join(','):'';
        this.applyOpt.inviterName = masterName&&masterName.length>0?masterName.join(','):'';
        setSessionStore('questionSubmitData',this.applyOpt);
        this.$router.back();
      },
      //选择师傅
      masterBtn(opt,n1,str){
        if(this.masterCheck.length==3 && !this[str][n1].isCur){
          this.$dialog.notify({mes: '最多只能选择3个师傅！', icon: 'error', timeout: 1200});
          return false;
        }
        this[str][n1].isCur = !this[str][n1].isCur;
        if(this[str][n1].isCur){
          var len = this.masterCheck.filter(function(data){
            return data.id == opt.id;
          });
          if(len.length==0){
            this.masterCheck.push(opt);
          }
        }else{
          let masterArr = this.masterCheck.filter(function(data){
            return data.id != opt.id;
          });
          this.masterCheck = masterArr;
        }
      },
      //删除师傅
      deleteMaster(opt,n){
        this.masterCheck.splice(n,1);
        //删除列表里边选中项
        for(var i=0;i<this.masterList.length;i++){
          if(this.masterList[i].id==opt.id){
            this.masterList[i].isCur = false;
            break;
          }
        }
        //删除搜索列表里边选中项
        for(var i=0;i<this.searchMasterList.length;i++){
          if(this.searchMasterList[i].id==opt.id){
            this.searchMasterList[i].isCur = false;
            break;
          }
        }
      },
      //搜索师傅列表
      fetchSearchList(searchKey){
        var that = this;
        this.$store.dispatch('log', {account:LOGCODE.INTERACTION.BTN_PAGE_MASTERWORKER_SEARCH});
        that.$http.get(this.apiSearchMaster+this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId+'&text='+searchKey)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.searchMasterList = data.result;
              this.$store.dispatch('log', {account:LOGCODE.INTERACTION.MESS_PAGE_MASTERWORKER_SEARCH_OK});
            }else {
              // that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          },(response) => {
            setTimeout(()=>{
              that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            },300);
          });
      },
      //获取师傅列表
      fecthMasterList(Pfile){
        var that = this;
        var _url = this.interactionOpt.token+'&memberId='+this.interactionOpt.memberId;
        this.$dialog.loading.open('加载中');
        that.$http.get(this.apiMasterUrl+_url)
          .then((response) => {
            const data = response.data;
            if(data.code == "s_ok" || data.code == "S_OK") {
              this.masterList = data.result;
              var _masterArr = this.applyOpt.inviterList.split(',');
              for(var i=0;i<this.masterList.length;i++){
                if(_masterArr.length>0){
                  var len = _masterArr.filter(function(data){
                    return data == that.masterList[i].id
                  });
                  that.masterList[i].isCur = len.length>0? true:false;
                }else{
                  that.masterList[i].isCur = false;
                }
                if(that.masterList[i].isCur){
                  this.masterCheck.push(that.masterList[i]);
                }
              } 
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