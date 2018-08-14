<template>
  <transition name="page">
    <section class="ck-doc store">
      <div class="store-bd search">
      	<div class="store-seach">
      		<i class="iconfont">&#xe602;</i>
      		 <input type="text" @focus="handleFocus" v-on:blur="changeInput" placeholder="请输入门店名称或品牌搜索" style="width: 80%;" v-model="searchKey"/>
      	</div>
        <div :class='{"all_ckeck_btn":isHasAll,"all_ckeck_btn all_hide":!isHasAll}' v-if="isShowAll"><div :class="{'gou':allChecked,'none':!allChecked}" @click='allCheckBox'>&nbsp;</div>全选</div>
      	<ul>
    			<li v-for="(store, index) in storeListTmp" @click="change(store)">			 
    				<div  :class="{'gou':store.isSelect,'none':!store.isSelect}">&nbsp;</div>
    				<span>{{store.orgShortName}}</span>
    			</li>
		    </ul>
        
      </div>
      <div :class='{"store-btn store-r" : isOpen,"store-btn" : !isOpen,"dis":isKeyUp}' @click="submit">确定</div>
    </section>
  </transition>
</template>
<style type="text/css">
  .registerrefer-bd .inner {
    position: relative;
    height: 0.86rem;
    line-height: 0.86rem;
    margin: 0;
    background-color: #fff;
    text-align: center;
    font-size: 85%;
  }
  .dis{
    position: static;
  }
  .all_hide {
    display: none;
  }
</style>
<script type="text/babel">

import { searchFromData, trim } from '../modules/filters';
import {removeSessionStore,setSessionStore,getSessionStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
import ajaxModel from '../utils/ajaxModel';
export default {
  data() {
    return {
      title: '选择门店',
      selected:false,
      selectId:'',
      isOpen: false,
      isKeyUp:false,
      searchKey:'',
      source:'',//页面调整来源
      isShowAll:true,
      allChecked:false,
      storeSelect:[],
      isHasAll:true,
      searchFlag:true,
      // storeApiUrl: window.baseUrl+window.appName+'/wxpay/listOrgs?orgCode='+window.orgCode,
      storeListTmp:[],      
      optStore:{
      	reqType:'apply'
      },
      storeList: [{'orgShortName':''}] 
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
            var filtersDatas = searchFromData(this.searchKey, this.storeList, 'orgShortName');
            this.storeListTmp = filtersDatas;
            this.setAllCheckebox();
          },1000);
        }
      }else {
        this.storeListTmp = this.storeList;
        this.setAllCheckebox();
      }      
    },
    'storeList'(){
      this.storeListTmp = this.storeList;
    }
  },
  mounted() {
    //初始化页面标题
    document.title = this.title;
    //挂载完毕初始化页面数据
    ajaxModel.listOrgs(this,this.fetchReferrerList);
    // this.fetchReferrerList();
    //拿缓存中选择的门店
    this.setStoreSelect();
    this.source = this.$route.query.from;
    if(this.source == 'glassOpenOrder' || this.source == 'recharge'){
      this.isShowAll = false;
    }
    this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_STORE});
  },
  methods: {
    fetchSearchRefer() {
      storeList
    },
    setStoreSelect(){
    	//从sessionStorage中加载已经勾选的信息
      var storeStr = getSessionStore('information_selected_store',true);
        if(typeof(storeStr) != 'boolean'){
          this.storeSelect = storeStr;
          if(this.storeSelect.length>0){
            this.isOpen = true;
          }
        } 
    },
    fetchReferrerList(result) {
      //获取地址列表请求方法
      let that = this;
      this.setStoreSelect();
      //这里使用双循环判断是因为要处理缓存中已经选择过的列表项
    	for(var i=0;i<result.length;i++){
    		var _i = result[i];
    		 result[i].isSelect=false;
    		for(var j=0;j<that.storeSelect.length;j++){
    			var _j = that.storeSelect[j];
    			if(_i.id==_j.id){
    				result[i].isSelect = true;
    			}
    		}          		
    	}
      that.storeList = result;
      if(that.storeList.length==that.storeSelect.length){
        that.allChecked = true;
      }
    },
    submit(){
      let that = this;
    	if(this.isOpen){
        this.storeSelect = this.storeListTmp.filter(function(data){
            return data.isSelect ==true;
        });
        setSessionStore("information_selected_store",this.storeSelect);
        if(this.source=='applyResult'){
          this.$router.push({
              path: '/'+this.source,
              query:{
                serverCode: this.$route.query.serverCode,
                serviceValue: this.$route.query.serviceValue,
                applyId : this.$route.query.applyId,
                type:'2'
              }
          });
        }else{
          this.$router.push({
              path: '/'+this.source
          });
        }
    	}else{
    		return;
    	}
    },
    handleFocus(){
      this.isKeyUp = true;
    },
    changeInput(){
       this.isKeyUp = false;
    },
    setAllCheckebox(){

      let c_len = this.storeListTmp.filter(function(data){
          return data.isSelect == false;
        });
       this.isHasAll =  true;
      if(c_len.length==0 && this.storeListTmp.length>0){
        this.allChecked = true;
      }else{
        this.allChecked = false;
        if(this.storeListTmp.length==0){
            this.isHasAll = false;
        }
      }
    },
    allCheckBox(){
        this.allChecked = !this.allChecked;
        if(this.allChecked){          
          this.isOpen=true;
        }else{
          this.isOpen=false;
        }
        this.storeSelect = [];
        let that = this;
        for(var i=0;i<this.storeList.length;i++){
          this.storeList[i].isSelect = this.allChecked;
          if(this.allChecked){
            this.storeSelect.push({'orgShortName':this.storeList[i].orgShortName,'id':this.storeList[i].id});
          }          
        }
    },
    change(pObj){
    	pObj.isSelect = !pObj.isSelect;
    	if(pObj.isSelect){
        if(this.source=='glassOpenOrder'|| this.source=='recharge'){//从开单跳转过来
          for(var i=0;i<this.storeList.length;i++){
            var item = this.storeList[i];
            this.storeList[i].isSelect = item.id==pObj.id? true:false;
          }
          this.storeSelect = [];
        }
        this.storeSelect.push({'orgShortName':pObj.orgShortName,'id':pObj.id});
        this.isOpen=true;
    	}else{
    		let _storeArr= this.storeSelect.filter(function(data){
    			return data.id != pObj.id;
    		});
    		this.storeSelect =  _storeArr;
    		if(this.storeSelect.length>0){
    			this.isOpen=true;
    		}else{
    			this.isOpen=false;
    		}
    	}
      if(this.storeSelect.length == this.storeListTmp.length){
        this.allChecked = true;
      }else{
        this.allChecked = false;
      }
    }

  }
}
  </script>
