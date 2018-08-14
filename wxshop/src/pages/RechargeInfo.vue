<template>
	<section class="ck-doc rechargeInfo">
    <ul>
      <li class="item">
        <div class="head"><i class="h_orderDetail">&nbsp;</i></div>
        <div class="content">
          <p class="order_code">订单编号：{{detailInfo.orderCode}}</p>
          <div class="item1 bor_b" v-if="detailInfo.wxUserParam">
            <div><span>姓名：</span>{{detailInfo.wxUserParam.name}}</div>
            <div><span>会员卡：</span>{{detailInfo.wxUserParam.vipCard}}</div>
            <div><span>手机号：</span>{{detailInfo.wxUserParam.mobilePhone}}</div>
            <div><span>组织：</span>{{detailInfo.orgShortName}}</div>
          </div>
          <div class="item1 bor_b" v-if="detailInfo.rechargePayload">
            <div><span>充值金额：</span>{{detailInfo.rechargePayload.credit}}</div>
            <div><span>赠送金额：</span>{{detailInfo.rechargePayload.exchange}}</div>
            <div><span>到账金额：</span>{{detailInfo.rechargePayload.total}}</div>
          </div>
          <div class="item1">
            <div>
              <span>微信支付：</span>
              <span class="red" v-if="detailInfo.wxStatus==0">待支付</span>
              <span class="red" v-else-if="detailInfo.wxStatus==1">支付成功</span>
              <span class="red" v-else-if="detailInfo.wxStatus==2">支付失败</span>
              <span class="red" v-else>已取消</span>
            </div>
            <div>
              <span>展达充值：</span>
              <span class="red" v-if="detailInfo.zdStatus==0">待支付</span>
              <span class="red" v-else-if="detailInfo.zdStatus==1">支付成功</span>
              <span class="red" v-else-if="detailInfo.zdStatus==2">支付失败</span>
              <span class="red" v-else>已取消</span>
            </div>
            <div><span>充值时间：</span>{{detailInfo.createTime}}</div>
          </div>
        </div>
      </li>
      <!-- 昨日充值统计 -->
      <li class="item re_count" v-show="rechargeInfo.rechargeDetails && rechargeInfo.rechargeDetails.length>0 && rechargeInfo.orgSize==1">
        <div class="head"><i class="h_rechargeCount">&nbsp;</i></div>
        <div class="content">
          <div class="item2" v-for="order in rechargeInfo.rechargeDetails">
            <div><span class="s_1">{{order.userName}}</span><span>充{{order.rechargeMoney}}送{{order.giveMoney}}</span></div>
            <div><span>{{order.rechargeTimeString.substring(11,16)}}</span></div>
          </div>
        </div>
      </li>
      <!-- 充值金额排名 -->
      <li class="item re_people" v-if="rechargeInfo.rechargeAmountRank && rechargeInfo.rechargeAmountRank.length>0 && rechargeInfo.orgSize>1">
        <div class="head"><i class="h_rechargeAmount">&nbsp;</i></div>
        <div class="content">
          <div class="item2 num1" v-for="(itemAmount,index) in rechargeInfo.rechargeAmountRank">
            <div>
              <span class="con one" v-if="(index+1) ==1">1</span>
              <span class="con two" v-else-if="(index+1) ==2">2</span>
              <span class="con three" v-else-if="(index+1)==3">3</span>
              <span class="ind" v-else>{{index+1}}</span>
              <span>{{itemAmount.orgName}}</span>
            </div>
            <div><span>￥{{itemAmount.rechargeAmount}}</span></div>
          </div>
        </div>
      </li>
      <!-- 充值订单排名 -->
      <li class="item re_people" v-if="rechargeInfo.rechargeOrdersRank && rechargeInfo.rechargeOrdersRank.length>0 && rechargeInfo.orgSize>1">
        <div class="head"><i class="h_rechargeOrder">&nbsp;</i></div>
        <div class="content">
          <div class="item2 num1" v-for="(itemOrder,index) in rechargeInfo.rechargeOrdersRank">
            <div>
              <span class="con one" v-if="(index+1) ==1">1</span>
              <span class="con two" v-else-if="(index+1) ==2">2</span>
              <span class="con three" v-else-if="(index+1)==3">3</span>
              <span class="ind" v-else>{{index+1}}</span>
              <span>{{itemOrder.orgName}}</span>
            </div>
            <div><span>{{itemOrder.orderCount}}</span></div>
          </div>
        </div>
      </li>
      <!-- 充值推荐人排名 -->
      <li class="item re_people" v-if="rechargeInfo.rechargeEmployees && rechargeInfo.rechargeEmployees.length>0">
        <div class="head"><i class="h_rechargePeople">&nbsp;</i></div>
        <div class="content">
          <div class="item2 num1" v-for="(itemPeople,index) in rechargeInfo.rechargeEmployees">
            <div style="width: 83%;">
              <span class="con one" v-if="(index+1) ==1">1</span>
              <span class="con two" v-else-if="(index+1) ==2">2</span>
              <span class="con three" v-else-if="(index+1)==3">3</span>
              <span class="ind" v-else>{{index+1}}</span>
              <span v-if="rechargeInfo.orgSize==1 || (rechargeInfo.orgSize>1 && itemPeople.orgName=='')" class="s_2">{{itemPeople.employeeName}}</span>
              <span v-else>{{itemPeople.employeeName}}<span class="s_3">（{{itemPeople.orgName}}）</span></span>
              <!-- <span class="s_5">{{itemPeople.employeeName}}<span class="s_3">(广州哈弗旗舰店)</span></span> -->
            </div>
            <div><span v-if="itemPeople.userCount && itemPeople.userCount!=0">{{itemPeople.userCount}}</span></div>
            <!-- <div><span>57000</span></div> -->
          </div>
        </div>
      </li>
      <!-- 昨日入会统计 (单门店)-->
      <li class="item re_count" v-if="rechargeInfo.userIntaitions && rechargeInfo.userIntaitions.length>0 && rechargeInfo.orgSize==1">
        <div class="head"><i class="h_menberCount">&nbsp;</i></div>
        <div class="content">
          <div class="item2" v-for="(itemUser,index) in rechargeInfo.userIntaitions">
            <div><span class="s_1">{{itemUser.userName}}</span><span class="p_24">会员卡：{{itemUser.cardNumber}}</span></div>
            <div><span>{{itemUser.initiationTimeString.substring(11,16)}}</span></div>
          </div>
        </div>
      </li>
      <!-- 昨日入会统计 (多门店)-->
      <li class="item re_count re_people"  v-show="rechargeInfo.memberToJoinRank && rechargeInfo.memberToJoinRank.length>0 &&  rechargeInfo.orgSize>1">
        <div class="head"><i class="h_menberCount">&nbsp;</i></div>
        <div class="content">
          <div class="item2" v-for="(itemUser,index) in rechargeInfo.memberToJoinRank">
            <div>
              <span class="con one" v-if="(index+1) ==1">1</span>
              <span class="con two" v-else-if="(index+1) ==2">2</span>
              <span class="con three" v-else-if="(index+1)==3">3</span>
              <span class="ind" v-else>{{index+1}}</span>
              <span class="s_4">{{itemUser.orgName}}</span>
            </div>
            <div><span>{{itemUser.memberCount}}</span></div>
          </div>
        </div>
      </li>
    </ul>
    <div class="point_out">&nbsp;</div>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../utils/assist';
  import {codeKeyValue} from '../utils/errorCode';
  import {LOGCODE} from '../utils/logCode';
  export default {
    data() {
      return {
        title: '充值详情',
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/report/wxPayDetail?sid='+window.shopsid,
        // apiUrl: window.baseUrl+window.appName+'/shopapi/func/report/wxPayDetail?sid=7F0000015E6E117E815E6EB2418A000F',
        rechargeInfo:{},
        detailInfo:{}
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchServiceDetail();
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_RECHARGEINFO});
      },
      fetchServiceDetail: function() {
        //获取首页数据
        let that = this;
        var did = window.location.href.split('rechargeInfo/')[1];
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl+'&did='+did)
        // that.$http.post(that.apiUrl+'&did=CZ15040935780203620')
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.rechargeInfo = data.var;
              that.detailInfo = data.var.detailInfo;
            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004") {
              this.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                this.$router.push('/login');
              }});
            }else if(data.code == "NO_LOGIN") {
              this.$dialog.notify({mes: '您还没有登录', icon: 'error', timeout: 1200, callback: () => {
                this.$router.push('/login');
              }});
            }else {
              this.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
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
      }
    }
  }
</script>