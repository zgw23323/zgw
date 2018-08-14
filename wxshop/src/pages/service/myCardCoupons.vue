<template>
	<section class="ck-doc myCardCoupons">
    <article class="head" style="display: none;">
      <div class="item"><span :class="{'cur':isType==1}" @click="menBtn(1)">线上卡券</span></div>
      <div class="item"><span :class="{'cur':isType==2}">线下卡券</span></div>
      <!-- <div class="item"><span :class="{'cur':isType==2}" @click="menBtn(2)">线下卡券</span></div> -->
    </article>
    <article class="content" style="padding-top: .2rem;">
      <ul>
        <li v-for='coupon in couponList'>
          <div class="l_left">
            <div><span class="c_money">￥{{coupon.couponAmount}}</span><span class="c_mess">满{{coupon.leastConsumpMoney}}元可用</span></div>
            <div class="c_title">{{coupon.couponName}}</div>
            <div>有效期至{{coupon.endTime}}</div>
          </div>
          <div class="l_right canUser" v-if="coupon.receiveCount>coupon.usedCount && coupon.couponStatus"  @click='goToUserCoupon(coupon)'>
            <span>去消费</span><i class="yuan yuan_top">&nbsp;</i><i class="yuan yuan_bottom">&nbsp;</i>
          </div>
          <div class="l_right noUser" v-else>
            <span>去消费</span><i class="yuan yuan_top">&nbsp;</i><i class="yuan yuan_bottom">&nbsp;</i>
            <i class="no_user_logo">&nbsp;</i>
          </div>
        </li>
      </ul>
    </article>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        title:'我的卡券',
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/serve/rightsListDriveDot?update=true',
        apiUserCouponListUrl: window.baseUrl+window.appName+'/shopapi/func/coupon/userCouponList?orgCode='+window.orgCode+'&shopsid='+window.shopsid, 
        infoDataList:[],
        isType: 1,
        showType:'',
        total:0,
        couponList:[],
        optApply: {
          curPage:1,
          pageSize:8,
          isRecharge:'',
          isShop:''
        }
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchCouponsData();
      window.addEventListener('scroll', this.menu);
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
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_SERVICECITY});
        document.body.scrollTop = document.documentElement.scrollTop = 0;
      },
      menu(){
        if ((document.documentElement.scrollHeight) == (document.documentElement.scrollTop | document.body.scrollTop) + document.documentElement.clientHeight){
            this.loadList();
        }
      },
      //选项卡点击
      menBtn(n){
        this.isType = n;
        this.optApply.curPage=1;
        this.fetchCouponsData();
      },
      //去使用优惠券
      goToUserCoupon(opt){
        var _url = '';
        if(opt.isRechargeCoupon==1 && opt.isShopCoupon == 0){ //跳去充值
            _url = '/recharge';
        }else{
            _url = '/goods';
        }
        this.$router.push({
            path: _url,
            query: {
              pId: 'coupon'
            }
        });
        sessionStorage.setItem('couponId',opt.couponId);
      },
      loadList() {
        //分页加载页面数据
        let that = this;
        that.optApply.curPage++;
        if(that.optApply.curPage > that.total) {
          /* 所有数据加载完毕 */
          that.optApply.curPage = that.total;
          return;
        }
        that.$http.post(this.apiUserCouponListUrl,this.optApply)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              for(var i=0;i<data.var.userCouponList.length;i++) {
                that.couponList.push(data.var.userCouponList[i]);
              }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            /* 单次请求数据完毕 */
            //window.$yduiBus.$emit('ydui.infinitescroll.finishLoad');

          },(response) => {
            that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
          });
      }, 
      fetchCouponsData: function() {
        //获取首页数据
        let that = this;
        // that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUserCouponListUrl,that.optApply)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.couponList = data.var.userCouponList;
              var currTime = Math.floor(new Date((data.var.currentTime).replace(/-/g,"\/")).getTime() / 1000);
              for(var i=0;i<that.couponList.length;i++){
                var item = that.couponList[i];
                  var endTime = Math.floor(new Date((item.endTime).replace(/-/g,"\/")).getTime() / 1000);
                  if(currTime<endTime && item.receiveCount>item.usedCount){
                    that.couponList[i].couponStatus = true;
                  }else{
                    that.couponList[i].couponStatus = false;
                  }
                that.couponList[i].startTime = item.startTime.split(' ')[0];
                that.couponList[i].endTime = item.endTime.split(' ')[0];
              }
              that.total = data.var.totalPage;
              if(that.couponList == 0){
                that.couponMess = '暂无可用优惠券';
              }
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>