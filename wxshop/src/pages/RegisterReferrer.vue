<template>
  <transition name="page">
    <section class="ck-doc registerrefer">
      <div class="registerrefer-bd">
      <div class="m-cell referer-group">

        <div class="cell-item referer-item">
          <div class="inner">
            <i class="iconfont">&#xe602;</i>
            <input type="text" placeholder="请输入搜索的推荐人姓名" v-model="searchKey"/>
          </div>
        </div>

        <div class="cell-item referer-item" @click="change(-1, {id:''})">

          <div class="cell-left">
              <span class="cell-icon"><slot name="icon"></slot></span>
              <slot name="left">
                <span slot="left" class="no-refer">无推荐人</span>
                <span slot="right"></span>
              </slot>
          </div>
          <div :class="{'cell-right':true,'re_select' : '' == selected}">
              <slot name="right"><i class="iconfont">&#xe72e;</i></slot>
              <slot name="i">&nbsp;</slot>
          </div>
        </div>
        <div class="cell-item referer-item" @click="change(index, referer)" v-for="(referer, index) in referrerListTmp">
          <div class="cell-left">
              <span class="cell-icon"><slot name="icon"></slot></span>
              <slot name="left">
                <span slot="left" class="vip-des">{{referer.realName}}</span>
                <span slot="right"></span>
              </slot>
          </div>
          <div :class="{'cell-right':true,'re_select' : referer.id == selected}">
              <slot name="right"><i class="iconfont">&#xe72e;</i></slot>
              <slot name="i">&nbsp;</slot>
          </div>
        </div>
      </div>
      </div>
    </section>
  </transition>
</template>
<style type="text/css">
  .registerrefer-bd .inner {
    width: 80%;
    text-align: left !important;
  }
  .registerrefer-bd .inner input {
    width: 80%;
  }
</style>
<script type="text/babel">

import { searchFromData, trim } from '../modules/filters';
import {setShopsId,removeSessionStore,setSessionStore,getSessionStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
import ajaxModel from '../utils/ajaxModel';
export default {
  data() {
    return {
      title: '入会推荐人',
      selected:'',
      searchKey:'',
      searchFlag:true,
      referrerApiUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxListEmployee',
      referrerListTmp:[],
      referrerList: [{"id":8,"realName":"服务顾问"}] //[{"id":8,"realName":"服务顾问"},{"id":9,"realName":"文兴"}]
    }
  },
  watch: {
    '$route.path': 'markInit',   //监听当前路由变化（辅助初始化）
    'searchKey'() {//监听输入搜索的关键字，间隔1秒匹配数据源
      let _searchKey = trim(this.searchKey,'g');
      if (_searchKey != "") {
        if(this.searchFlag) {
          this.searchFlag = false;
          setTimeout(()=>{
            this.searchFlag = true;
            var filtersDatas = searchFromData(this.searchKey, this.referrerList, 'realName');
            this.referrerListTmp = filtersDatas;
          },1000);
        }
      }else {
        this.referrerListTmp = this.referrerList;
      }
    },
    'referrerList'(){
      this.referrerListTmp = this.referrerList;
    }
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
    ajaxModel.wxListEmployee(this,this.fetchReferrerList);
  },
  methods: {
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      
      setShopsId();
      var query = this.$route.query.from;
      if (query == 'recharge') {  //从充值页面跳转
        let selectedIndex = sessionStorage.getItem("recharge_select_referrer_index");
        if (selectedIndex != null && selectedIndex != undefined) {
          this.selected = selectedIndex;
        }
        this.$store.dispatch('log', {account: LOGCODE.HOME.PAGE_RECHARGE_REGISTERREFERER});
      }else if(query == 'glassClaim' || query == 'glassOpenOrder'){
        let selectedOpt = getSessionStore("glass_select_referrer",true);
        if (typeof(selectedOpt) != 'boolean') {
          this.selected = selectedOpt.id;
          console.log('selected',this.selected,selectedOpt);
        }
      }else {
        let selectedIndex = sessionStorage.getItem("_select_referrer_index");
        if (selectedIndex != null && selectedIndex != undefined) {
          this.selected = selectedIndex;
        }
        this.$store.dispatch('log', {account: LOGCODE.HOME.PAGE_REGISTER_REGISTERREFERER});
      }
    },
    fetchSearchRefer() {
      referrerList
    },
    change(i, referObj) {
      //切换地址   
      var query = this.$route.query.from;
      if (query == 'recharge') {//会员充值页面跳转而来
        sessionStorage.setItem("recharge_select_referrer_index", referObj.id);
        sessionStorage.setItem("recharge_select_referrer", JSON.stringify(referObj));
        this.$router.back();
      }else if(query == 'glassOpenOrder' || query == 'glassClaim'){//从玻璃险开单页面跳转
        sessionStorage.setItem("recharge_select_referrer_index", referObj.id);
        sessionStorage.setItem("glass_select_referrer", JSON.stringify(referObj));
        if(query == 'glassClaim'){
          var opt = {type: 'update',id: this.$route.query.id};
          if(this.$route.query.openId){
            opt.openId = this.$route.query.openId;
          }
          this.$router.push({
            path: '/glassClaim',
            query: opt
          });
        }else{
          this.$router.push({
            path: '/glassOpenOrder',
            query: {type:'2',messType:this.$route.query.messType}
          });
        }
      }else {//从会员注册页面跳转
        sessionStorage.setItem("_select_referrer_index", referObj.id);
        sessionStorage.setItem("_select_referrer", JSON.stringify(referObj));
        this.$router.back();
      }

    },
    fetchReferrerList(result) {
      //获取推荐人列表请求方法
      // this.referrerList = resultArr;
      // this.$dialog.loading.close();
      let that = this;
      that.referrerList = result;
    }
  }
}
  </script>
