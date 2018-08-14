<template>
  <section class="ck-doc coupon">
    <div v-if="!tips2">
      <div class="filterbar-wrap" v-if="!queryType">
        <div class="filter-bar">
          <ul class="sort-tab">
            <li class="sort" @click="dropList(1)" :class="{'current': dropList1, 'selected': 1 == m1navcurId ? true : false}">按过期时间<i class="arrow"></i></li>
            <li class="sort" @click="dropList(2)" :class="{'current': dropList2, 'selected': 2 == m1navcurId ? true : false}">按开始时间<i class="arrow"></i></li>
            <li class="sort common-sort" @click="dropList(3)" :class="{'current': dropList3, 'selected': 3 == m1navcurId ? true : false}"><span>按金额</span><i class="arrow"></i></li>
          </ul>
        </div>
      </div>
      <div :class='{"coupon-list":!queryType,"coupon-person":queryType}'>
        <yd-coupon-item slot="list" :typeCoupon="'list'" :couponData="couponList" v-if="couponList.length>0"></yd-coupon-item>
        <div v-if="couponList.length==0" class="no_coupon">{{couponMess}}</div>
      </div>
    </div>
    <yd-alert-tip :alertText="tipText" :tipsType='"tips1"' v-if='tips1'></yd-alert-tip>
    <div class="tipsPage" v-if="tips2">
      <div class="tipsBox">
        <div class="tipStatus" data-type="busy"></div>
        <p>亲，网络出问题啦~~</p>
        <div class="btns" @click="reLoad">重新加载</div>
      </div>
    </div>

  </section>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
export default { 
    data() {
      return {
          title: '优惠券',
          disabled: true,
          tips2: false,
          tips1:false,
          // isLoading:false,
          isCallback:false,
          curDrop:1,
          queryType :false,
          dropList1: false,
          dropList2: false,
          dropList3: false,
          m1navcurId:0,
          couponMess:'',
          couponList:[],
          tipText:'亲，此限购商品每人只能购买<span class="red">8</span>件',
          apiShopCouponUrl: window.baseUrl+window.appName+'/shopapi/func/coupon/shopCouponList?orgCode='+window.orgCode,
          apiUserCouponListUrl: window.baseUrl+window.appName+'/shopapi/func/coupon/userCouponList?orgCode='+window.orgCode+'&shopsid='+window.shopsid, 
          opts: {
            curPage: 1, 
            pageSize: 6,
            sortType:0,
            couponType:-1,
            isDesc:1
          },
          userOpt:{ 
            curPage:1,
            pageSize:6,
            isRecharge:'',
            isShop:''
          },
          couponTotal:0,
          isDesc:0//当前排序、1=降序，2=升序         
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchCoupontList();
      window.addEventListener('scroll', this.menu);
    },
    watch: {
      '$route.path': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    methods: {
      reLoad() {
        //重载页面
        this.$router.go(0);
        this.markInit();
      },
      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
            this.loadList();
        }
      },
      markInit() {
        //初始化页面标题
        if(this.$route.query.mark == 'person'){
          document.title = '我的优惠券';
          this.queryType = true;
        }else{
          this.queryType = false;
        }

        //获取shopsid缓存数据
        setShopsId();
        document.body.scrollTop = document.documentElement.scrollTop = 0;
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_COUPON});
      },
      loadList() {
        //分页加载异步请求数据
        let that = this;
        if(this.$route.query.mark == 'person'){//获取我的优惠券列表
          that.userOpt.curPage++;
          if(that.userOpt.curPage > that.couponTotal) {
            /* 所有数据加载完毕 */
            that.userOpt.curPage = that.couponTotal;
            return;
          }
        }else{//获取优惠券列表
          that.opts.curPage++;
          if(that.opts.curPage > that.couponTotal) {
            /* 所有数据加载完毕 */
            that.opts.curPage = that.couponTotal;
            return;
          }
        }
        that.fetchCoupontList('loading');
      },
      //获取优惠券列表
      fetchCoupontList(str){
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        if(this.$route.query.mark == 'person'){//获取我的优惠券列表
          var _api = that.apiUserCouponListUrl;
          var _opts = that.userOpt;
        }else{//获取优惠券列表
          var _api = that.apiShopCouponUrl;
          var _opts = that.opts;
        }
        that.$http.post(_api,_opts)
          .then((response) => {
            const data = response.data;
            // that.tips1 = false;
            if(data.code == "S_OK") {
              if(this.$route.query.mark == 'person'){//获取我的优惠券列表
                var couList = data.var.userCouponList;
              }else{//获取优惠券列表
                var couList = data.var.couponList;
                that.isDesc = that.opts.isDesc;
              }

              if(str == 'loading'){
                for (let list of couList) {
                  that.couponList.push(list);
                }
              }else{
                that.couponList = couList;
              }
              if(this.$route.query.mark == 'person'){
                var currTime = Math.floor(new Date((data.var.currentTime).replace(/-/g,"\/")).getTime() / 1000);
              }
           
              for(var i=0;i<that.couponList.length;i++){
                var item = that.couponList[i];
                
                if(this.$route.query.mark == 'person'){
                  var endTime = Math.floor(new Date((item.endTime).replace(/-/g,"\/")).getTime() / 1000);
                  if(currTime<endTime && item.receiveCount>item.usedCount){
                    that.couponList[i].couponStatus = true;
                  }else{
                    that.couponList[i].couponStatus = false;
                  }
                }
                that.couponList[i].startTime = item.startTime.split(' ')[0];
                that.couponList[i].endTime = item.endTime.split(' ')[0];
              }
              that.couponTotal = data.var.totalPage;
              if(that.couponList == 0){
                that.couponMess = '暂无可用优惠券';
              }
            }else{
              if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN"){
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else{
                that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
              }
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips2 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      //优惠券筛选
      dropList(pType){
        this.opts.sortType = pType;
        console.log({"ty":this.m1navcurId,'t2':this.isDesc});
        if(this.m1navcurId == pType){//点击项是当前选中项
          if(this.isDesc ==1){//改变搜索顺序，正序改为倒序
            this.opts.isDesc = 2;
          }else{
            this.opts.isDesc = 1;
          }
        }else{
          this.m1navcurId = pType;
          this.opts.isDesc = 1;
          if(pType==1){
            this.dropList1 = true;
            this.dropList2 = false;
            this.dropList3 = false;
          }else if(pType==2){
            this.dropList1 = false;
            this.dropList2 = true;
            this.dropList3 = false;
          }else{
            this.dropList1 = false;
            this.dropList2 = false;
            this.dropList3 = true;
          }
        }
        console.log(this.opts);
        this.opts.curPage = 1;
        this.fetchCoupontList();
      }
    }
}
</script>
