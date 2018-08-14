<template>
<!-- <yd-layout> -->

    <section class="ck-doc storeAccounting">
      <article class="head">
        <div class="item"><span :class="{'cur':isType==1}" @click="menBtn(1)">玻璃保障购买</span></div>
        <div class="item"><span :class="{'cur':isType==2}">玻璃保修服务</span></div>
      </article>
      
        <article class="content" >
          <ul>
            <li v-if="storeOpt && storeOpt.openRecordId" @click="goTo(storeOpt)">
                <div class="left">
                  <p class="p1"><span>{{storeOpt.customerName}}</span><span>￥{{storeOpt.insurancePrice}}</span></p>
                  <p class="p2" v-if="storeOpt.createTime">{{storeOpt.createTime.substring(5,16)}}</p>
                </div>
                <div class="right">
                  <span class="yi" v-if="storeOpt.financeStatus==2">已确认</span>
                  <span class="dai" v-else>待确认</span>
                  <i class="right_icon">&nbsp;</i>
                </div>
              </li>

            <li v-for="item in accountList" @click="goTo(item)" :id="item.openRecordId" v-if="item.openRecordId != storeOpt.openRecordId">
              <div class="left">
                <p class="p1"><span>{{item.customerName}}</span><span>￥{{item.insurancePrice}}</span></p>
                <p class="p2" v-if="item.createTime">{{item.createTime.substring(5,16)}}</p>
              </div>
              <div class="right">
                <span class="dai" v-if="item.financeStatus==1 || item.financeStatus==null">待确认</span>
                <span class="yi" v-if="item.financeStatus==2">已确认</span>
                <i class="right_icon">&nbsp;</i>
              </div>
            </li>
          </ul>
        </article>
      
    </section>
  
  <!-- </yd-layout> -->
</template>
<script type="text/babel">
import { mapState } from 'vuex';
import {removeSessionStore,setSessionStore,getSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
export default {
  data() {
    return {
      title: '门店财务',
      isType: 1,
      opt: {
        curPage:1
      },
      storeOpt: {},
      hasNext:true,
      accountList:[],
      storeApiUrl: window.baseUrl+window.appName+'/shopapi/func/insurance/wxFinanceOpenList?openId='+window.openId//门店列表
    }
  },
  watch: {
    '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
  },
   
  methods: {
    markInit() {
        //初始化页面标题
        document.title = this.title;
        this.fetchMain(); 
        //获取shopsid缓存数据
        window.addEventListener('scroll', this.menu);
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_STOREACCOUNTING});
      },
    menu(){
      if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
        if(this.hasNext){
          this.loadList();
          this.isLoading = true;
        }else{
          this.isLoading = false;
        }
      }
    },
    loadList() {
        //分页加载异步请求数据
        let that = this;
        that.opt.curPage++;
        that.$http.post(that.storeApiUrl,this.opt)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(data.var.length){
                for (let list of data.var) {
                  that.accountList.push(list);
                }
                that.hasNext = true;
              }else{
                that.hasNext = false;
              }
            }else {
              that.$dialog.notify({mes: '数据加载失败，请稍后再试！', icon: 'error', timeout: 1200});
            }

            /* 单次请求数据完毕 */
            // that.$yduiBus.$emit('ydui.infinitescroll.finishLoad');

            // that.optsCart.curPage++;
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      },
    goTo(item) {
      item.isSubmit = item.financeStatus==2 ? true:false;
      setSessionStore('openRecordId_mess',item);
      this.$router.push({
        path:'/storeAccounDetail',
        query: {openRecordId: item.openRecordId}
      })
    },
    goToScroll(){
      var _openRecordOpt = getSessionStore('openRecordId_mess',true);
      if(typeof(_openRecordOpt) !="boolean"){
        this.storeOpt = _openRecordOpt;
      }
    },
    fetchMain(){
      let that = this;
      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.storeApiUrl,this.opt)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.accountList = data.var;
            }else{
              that.$dialog.notify({mes: '数据加载失败，请稍后再试！', icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
              that.goToScroll();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
    }
  },
  beforeDestroy() {
    //在页面销毁时（退出时）保存购物车数据
    window.removeEventListener('scroll', this.menu);
  }
}
</script>
