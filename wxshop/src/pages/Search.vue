<template>
  <transition name="page">
    <section class="ck-doc search">

      <div class="ck-search">
        <div style="height: .2rem"></div>
        <div class="inner">
          <i class="iconfont">&#xe602;</i>
          <input type="text" placeholder="请输入搜索的商品名" v-model="searchKey"/>
          <span class="btn-cancel" @click="routerBack">取消</span>
        </div>
        <ul class="search-input-list" v-if="searchList">
          <li v-for="list in searchListData" @click="searchGo(list.prodTitle)">{{list.prodTitle}}</li>
        </ul>
      </div>

      <div class="search-history" v-if="historyList">
        <h2 class="title">历史记录</h2>
        <ul class="search-history-list">
          <li v-for="list in historyListData" @click="searchGo(list.searchKey)">{{list.searchKey}}</li>
        </ul>
      </div>

    </section>
  </transition>
</template>

<script>
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
  export default {
    data() {
      return {
        title: '搜索',
        searchKey: '',
        searchList: true,
        historyList: true,
        searchListData: {},
        historyListData: {},
        apiSearchUrl: window.baseUrl+window.appName+'/shopapi/func/product/queryProductList?orgCode='+window.orgCode,
        apiHistoryUrl: window.baseUrl+window.appName+'/shopapi/func/search/ListSearch?orgCode='+window.orgCode,
        optsSearch: {pageSize: 8},
        optsHistory: {curPage: 1, pageSize: 10},
        searchFlag: true
      }
    },
    watch: {
      '$route.path': 'markInit',
      searchKey() { //实时联动搜索关键词
        let _searchKey = this.trim(this.searchKey,'g');
        if(_searchKey != "") {
          if(this.searchFlag) {
            this.searchFlag = false;
            this.fetchSearchList(_searchKey);
            setTimeout(()=>{
              this.searchFlag = true;
            },3000);
          }
        }else {
          this.searchListData = {};
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      document.title = this.title;
      this.fetchHistoryList();
    },
    methods:{
      markInit() {
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_SEARCH});
      },
      searchGo(str) {
        //根据关键词跳去商品列表（关键词筛选商品列表）
        this.$router.push({
            path: '/goods',
            query: {
              searchKey: str
            }
        });
      },
      routerBack(){
        //返回上一页
        this.$router.back();
      },
      trim(str,is_global) {  //去除空格方法（格式化字符串）
        let result;
        result = str.replace(/(^\s+)|(\s+$)/g,"");
        if(is_global.toLowerCase()=="g") {
            result = result.replace(/\s/g,"");
         }
        return result;
      },
      fetchSearchList(searchKey) { //通过关键字联动获取搜索列表
        let that = this;
        that.optsSearch.searchKey = that.searchKey;
        that.$http.post(that.apiSearchUrl+'&shopsid='+window.shopsid,that.optsSearch)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.searchListData = data.var;
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
          });
      },
      fetchHistoryList() { //获取历史搜索列表
        let that = this;
        that.$http.post(that.apiHistoryUrl+'&shopsid='+window.shopsid,that.optsHistory)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.historyListData = data.var;
            }else {
              //that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
          });
      }
    }
  }
</script>
