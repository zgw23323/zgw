<template>
	<transition name="page">
	<yd-layout>
    <section class="ck-doc">
    <yd-infinitescroll :on-infinite="loadList">

      <yd-list slot="list">
  		<div class="obligations-group">
  			<div class="obligations-item" v-for="(list,index) in orderList">
  				<div class="obligations-item-hd">
  					<span v-if="type2" class="s1">
              {{list.createTimeString}}
              <!-- <span>订单编号：{{list.orderNumber}}</span> -->
              <!-- 距离结束时间：<yd-countdown class="s1" :cursystem="list.systemTime" :time="list.orderDeadTimeString" format=" {%h}:{%m}:{%s}"></yd-countdown> -->
            </span>
  					<span v-if="type1" class="s1">{{list.createTimeString}}</span>
  					<span v-if="type1">
              <span class="tips2" v-if="list.orderStatus == 2">已完成</span>
              <span class="tips" v-if="list.orderStatus == 3">已取消</span>
              <span class="tips" v-if="list.orderStatus == 4">发货中</span>
              <span class="tips" v-if="list.orderStatus == 5">已发货</span>
              <!-- <span class="tips2" v-if="list.transactStatus == 1">交易成功</span>
              <span class="tips" v-if="list.transactStatus == 2">交易失败</span>
              <span class="tips" v-if="list.transactStatus == 0 && list.orderStatus==3">订单取消</span> -->
            </span>
            <span v-if="type2" class="tips2">待付款</span>
  				</div>
  				<div class="obligations-item-bd">
  					<ul class="obligations-item-list">
  						<li v-for="sub in list.detailList">
  							<div class="cl">
                  <div class="obligations-goods-img imgLazy" v-lazy:background-image="imgBaseUrl+sub.prodPicUrl" @click="detailsGo(sub.prodId,list)"></div>
                  <div class="obligations-goods-mes">
                    <p class="obligations-goods-title">{{sub.prodName}}</p>
                    <p class="od-goods-format">{{sub.prodGuigeDesc}}</p>
                    <p>
                      <span class="obligations-goods-price" v-if="list.fromSource==2">￥{{sub.spikePrice}}</span>
                      <span class="obligations-goods-price" v-else-if="list.fromSource==3">￥{{sub.discountPrice}}</span>
                      <span class="obligations-goods-price" v-else>￥{{sub.memberPrice}}</span>
                    </p>
                  </div>
                </div>
                <span class="obligations-goods-count">×{{sub.prodCount}}</span>
  						</li>
  					</ul>
  				</div>
  				<div class="obligations-item-ft cl" v-if="type2">
  					<div v-if="!list.nowStatus" class="btn fr" @click="pay(list.id,list)">支付</div>
            <div v-if="list.nowStatus" class="btn fr gray">支付</div>
  					<div class="cancel_btn" @click="cancleOrder(list.id)">取消订单</div>
            <div class="cancel_btn" @click="details(list.id,list)">查看订单</div>
  				</div>
  				<div class="obligations-item-ft cl" v-if="type1">
            <span class="btn fr" @click="details(list.id,list)">查看订单</span>
            <span class="cancel_btn" @click="removeOrder(list.id)">移除</span>
            <!--<div class="btn fr" @click="details(list.id)" v-if="list.transactStatus == 1">查看详情</div>
            <div class="btn fr" @click="payErrorGo" v-if="list.transactStatus == 2">查看详情</div>
            <div class="btn fr" @click="payErrorGo" v-if="list.transactStatus == 0">查看详情</div>-->
  					<!-- <div class="btn fr gray" @click="removeOrder(list.id)">移除</div> -->
  				</div>
  			</div>
  		</div>
      </yd-list>
      <!-- 数据全部加载完毕显示 -->
      <!--<span slot="doneTip">啦啦啦，啦啦啦，没有数据啦~~</span>-->
      <span slot="doneTip"></span>
      <!-- 加载中提示 -->
      <div slot="loadingTip" class="loadingTip"></div>
    </yd-infinitescroll>

    <div class="tipsPage" v-if="tips1">
      <div class="tipsBox">
        <div class="tipStatus" data-type="order"></div>
        <p>亲，您没有相关订单噢~~</p>
        <router-link to="/goods" class="btns">去逛逛</router-link>
      </div>
    </div>

		</section>
    </yd-layout>
	</transition>
</template>

<script type="text/babel">
import {setShopsId} from '../utils/assist';
import {codeKeyValue} from '../utils/errorCode';
import {LOGCODE} from '../utils/logCode';
export default {
    data() {
        return {
        	title: '待付款',
          start1: false,
          tips1: false,
        	type1: false,
        	type2: false,
        	imgBaseUrl: window.imgBaseUrl,
        	apiOrderListUrl: window.baseUrl+window.appName+'/shopapi/func/product/queryOrderList?orgCode='+window.orgCode+'&shopsid='+window.shopsid,
        	optsOrderList: {
            curPage: 1,
            pageSize: 5,
        		queryType: 2
        	},
        	orderList: [],
        	apiCancleOrderUrl: window.baseUrl+window.appName+'/shopapi/func/product/cancleOrder?orgCode='+window.orgCode+'&shopsid='+window.shopsid,
        	optsCancleOrder: {
        		orderIds: []
        	},
        	apiRemoveOrderUrl: window.baseUrl+window.appName+'/shopapi/func/product/removeOrder?orgCode='+window.orgCode+'&shopsid='+window.shopsid,
        	optsRemoveOrder: {
        		orderIds: []
        	}
        }
    },
    watch: {
      '$route.query.type': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
    	this.markInit();
    	this.fetchOrderList();
    },
    methods: {
      status(s) { //支付状态切换方法
      	let _s = "";
      	if(s == 1) {
      		_s = "付款 ";
      	}else if(s == 2) {
      		_s = "已完成";
      	}else if(s == 3) {
      		_s = "已取消";
      	}
      	return _s;
      },
      markInit() {
        //初始化页面标题，并且通过路由type切换（全部订单/待支付订单）对应显示样式
      	if(this.$route.query.type == 1) {
      		document.title = '全部订单';
      		this.type1 = true;
      		this.type2 = false;
      	}else {
      		document.title = this.title;
      		this.type2 = true;
      		this.type1 = false;
      	}
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_OBLIGATIONS});
      },
      //倒计时结束回调
      countdownBack(){
        // console.log('11');
        for(var i=0;i<this.orderList.length;i++){
          var item = this.orderList[i];            
          var currTime = Math.floor(new Date().getTime() / 1000); 
          var endTime = Math.floor(new Date((item.orderDeadTime).replace(/-/g,"\/")).getTime() / 1000);
          // console.log({'curr':currTime,'end':endTime});
          if(currTime>=endTime){
            this.orderList[i].nowStatus = true;
          }else{
            this.orderList[i].nowStatus = false;
          }
        }
      },
      detailsGo(id,pOpt) {
        if(pOpt.fromSource != 5){
          //跳去详情
          this.$router.push({
            path: '/details',
            query: {
              pid: id
            }
          });
        }else{
          //跳去一元拍商品详情
          this.$router.replace({
            path: '/actionProductDetail',
            query: {
              pid: id     
            }
          });
        }
      },
      payErrorGo() {
        //跳去支付失败页面
        this.$router.push('/payerror');
      },
      loadList() {
          //分页加载数据
          let that = this;
          that.optsOrderList.curPage++;
          if(that.optsOrderList.curPage > that.optsOrderList.total) {
            /* 所有数据加载完毕 */
            that.optsOrderList.curPage = that.optsOrderList.total;

            window.$yduiBus.$emit('ydui.infinitescroll.loadedDone');
            return;
          }
          that.start1 = false;
          that.$http.post(that.apiOrderListUrl+'&shopsid='+window.shopsid,that.optsOrderList)
            .then((response) => {
              const data = response.data;
              if(data.code == "S_OK") {
                var _list = data.var;
                if(that.$route.query.type==2){
                  for(var i=0;i<_list.length;i++){
                    var item = _list[i];
                    var currTime = Math.floor(new Date((item.systemTime).replace(/-/g,"\/")).getTime() / 1000); 
                    var endTime = Math.floor(new Date((item.orderDeadTime).replace(/-/g,"\/")).getTime() / 1000);
                    if(currTime>=endTime){
                      data.var[i].nowStatus = true;
                    }else{
                      data.var[i].nowStatus = false;
                    }
                  }
                }
                that.orderList = [...that.orderList, ...data.var];
                that.start1 = true;
              }else {
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }

              /* 单次请求数据完毕 */
              window.$yduiBus.$emit('ydui.infinitescroll.finishLoad');

              that.optsOrderList.curPage++;
            });
      },
      fetchOrderList() {
        //获取订单列表数据
      	let that = this;
      	that.optsOrderList.queryType = this.$route.query.type;
        //that.start1 = false;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiOrderListUrl,that.optsOrderList)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              var _list = data.var;
              if(that.$route.query.type==2){
                for(var i=0;i<_list.length;i++){
                  var item = _list[i];
                  var currTime = Math.floor(new Date((item.systemTime).replace(/-/g,"\/")).getTime() / 1000); 
                  var endTime = Math.floor(new Date((item.orderDeadTime).replace(/-/g,"\/")).getTime() / 1000);
                  if(currTime>=endTime){
                    data.var[i].nowStatus = true;
                  }else{
                    data.var[i].nowStatus = false;
                  }
                }
                // that.orderList = data.var;
              }
              that.orderList = data.var;
              that.optsOrderList.total = data.total;
              that.start1 = true;
              if(that.orderList.length == 0) {
                that.tips1 = true;
              }else {
                that.tips1 = false;
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
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      details(id,pOpt) {
        if(pOpt.fromSource != 5){
          //跳去订单结果详情
          this.$router.push({
            path: '/orderInfo',
            query: {
              oid: id
            }
          });
        }else{
          //跳去一元拍支付详情
          this.$router.replace({
            path: '/actionOrderInfo',
            query: {
              id: id
            }
          });
        }
      },
      pay(id,pOpt) {
        if(pOpt.fromSource != 5){
          //跳去支付详情页面
          this.$router.push({
              path: '/orderdetails',
              query: {
                source: 2,
                oid: id
              }
          });
        }else{//一元抢拍去支付
          this.$router.push({
              path: '/actionProductDetail',
              query: {
                pid: pOpt.detailList[0].prodId,
                orderNumber: pOpt.orderNumber,
                orderId: id
              }
          });
        }
      },
      cancleOrder(id) {
        //取消订单
      	let that = this;
        that.optsOrderList.curPage = 1;
      	that.optsCancleOrder.orderIds = [];
      	that.optsCancleOrder.orderIds[0] = { "orderId": id };
        that.$http.post(that.apiCancleOrderUrl,that.optsCancleOrder)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.$dialog.notify({mes: '取消订单成功', icon: 'error', timeout: 1000});
              that.fetchOrderList();
              this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_OBLIGATIONS_CANCLEORDER_OK});
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
          });
      },
      removeOrder(id) {
        //移除订单
        this.$dialog.confirm({
          title: '确认移除订单？',
          mes: '移除后无法恢复订单信息',
          opts: () => {
          	let that = this;
            that.optsOrderList.curPage = 1;
          	that.optsRemoveOrder.orderIds = [];
          	that.optsRemoveOrder.orderIds[0] = { "orderId": id };
            that.$http.post(that.apiRemoveOrderUrl,that.optsRemoveOrder)
              .then((response) => {
                const data = response.data;
                if(data.code == "S_OK") {
                  that.$dialog.notify({mes: '移除订单成功', icon: 'error', timeout: 1000});
                  that.fetchOrderList();
                  this.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_OBLIGATIONS_REMOVEORDER_OK});
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
              });
          }
        });
      }
  }
}
</script>
