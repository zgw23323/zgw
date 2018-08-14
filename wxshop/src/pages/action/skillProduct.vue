<template>
  <div>
    <section class="ck-doc skillProduct">
      <div v-if="!tips1" class="cl">
        <article class="head">
          <div><img src="../../img/actionImg/skill_logo.png"></div>
          <div class="d_1">
            <!-- <span>距本场结束</span> -->
            <!-- <span>00:20:30</span> -->
            <span class="t_mess" v-if="skillStatu.statu==3">距本场结束</span>
            <span class="t_mess" v-if="skillStatu.statu==1">距开始</span>
            <span class="t_mess" v-if="skillStatu.statu==4">已结束</span>
            <yd-countdown class="s1" :cursystem="skillStatu.currentTime" :callback="countDownCall" :time="skillStatu.endTime" :format="'{%h}:{%m}:{%s}'" v-if="skillStatu.statu==3"></yd-countdown>
            <yd-countdown class="s1" ref='countDown' :cursystem="skillStatu.currentTime" :callback="countDownCall" :time="skillStatu.startTime" :format="'{%h}:{%m}:{%s}'" v-if="skillStatu.statu==1"></yd-countdown>
          </div>
        </article>
        <article class="content">
          <ul>
            <li class="flex" v-for="item in skillProduct" @click="goTo(item.id,item.activityId,item.serveCode,2)">
              <div class="left"><img :src='imgBaseUrl+item.prodPic'></div>
              <div class="right">
                <div class="space">{{item.prodTitle}}</div>
                <div class="d_1">
                  <span class="s_1">限时价:
                    <span class="s_2" v-if="item.skipe==true">￥<span>{{item.skipePrice}}</span></span>
                    <span class="s_2" v-else>￥<span>{{item.discountPrice}}</span></span>
                  </span>
                  <span class="list-del-price">原价:￥{{item.orgPrice}}</span>
                </div>
                <div class="i_flex_betwent">
                  <div class="count_mess">
                    <span v-if="!item.initStocks" :style="{'width': item.stocksCount + '%','max-width': '100%'}">&nbsp;</span>
                    <span v-else-if="item.stocksCount>0" :style="{'width': ((item.stocksCount/item.initStocks)*100).toFixed(2) + '%','max-width': '100%'}">&nbsp;</span>
                    <span v-else :style="{'width': '100%','max-width': '100%'}">&nbsp;</span>
                  </div>
                  <span class="s_count">剩余{{item.stocksCount}}</span>
                  <div class="p_mess cur" v-if="item.status==3">马上抢</div>
                  <div class="p_mess" v-if="item.status==1">{{item.curDate}}开抢</div>
                  <div class="p_mess" v-if="item.status==4">已结束</div>
                </div>
              </div>
            </li>
          </ul>
        </article>
      </div>

      <div class="tipsPage" v-if="tips1">
        <div class="tipsBox">
          <div class="tipStatus" data-type="busy"></div>
          <p>亲，网络出问题啦~~</p>
          <div class="btns" @click="reLoad">重新加载</div>
        </div>
      </div>

    </section>
  </div>
</template>

<script type="text/babel">
import {setShopsId,getSessionStore,removeSessionStore,setSessionStore} from '../../utils/assist';
import {codeKeyValue} from '../../utils/errorCode';
import {LOGCODE} from '../../utils/logCode';
import ajaxModel from '../../utils/ajaxModel';
export default {
    data() {
        return {
          tips1: false, 
          title: '商品',
          imgBaseUrl: window.imgBaseUrl,
          opt:{
            curPage:1,
            pageSize:6,
            identifierCode:2,
            name:'秒杀'
          },
          skillStatu: {},//秒杀的状态
          skillProduct:[]//秒杀商品
        }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    beforeMount() {//fetchList('m3',menu.code,menu.name)
      //挂载完毕初始化页面数据
      ajaxModel.queryProductList(this,this.fectchQueryFun,this.opt,2);
      this.markInit();
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
        this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_GOOD});
      },
      countDownCall(){
        if(this.skillStatu.statu==3){
          this.skillStatu.statu=4;
        }else if(this.skillStatu.statu==1){
          this.isCoundownCall = true;
          ajaxModel.queryProductList(this,this.fectchQueryFun,this.opt,2);
        }
      },
      //秒杀商品跳去详情页
      goTo(id,activityId,serveCode,prodSource) {
        //跳去详情页
        if(serveCode == ''){
          this.$router.push({
            path: '/details',
            query: {
              pid: id,
              psource:prodSource,
              activityId:activityId
            }
          });
        }else{
          var str = serveCode.split('_')[0];
          this.$router.push({
            path: '/serviceDetail',
            query: {
              'pid': id,
              'serveCode':str,
              activityId:activityId,
              'psource':this.source
            }
          });
        }
      },
      fectchQueryFun(result){
        this.skillProduct = result;
        console.log('result',result);
        var _statu = this.skillProduct.filter((data)=>{
          return data.status == 3;//抢购中
        });
        if(_statu.length){
          this.skillStatu.statu = 3;
          this.skillStatu.currentTime =_statu[0].currentTime;
          this.skillStatu.endTime = _statu[0].endTime;
          // this.skillStatu.currentTime = '2018-05-17 16:55:00';
          // this.skillStatu.endTime = '2018-05-17 16:56:00';
        }else{
          _statu = this.skillProduct.filter((data)=>{
            return data.status == 1;//未开始、预约中
          });
          if(_statu.length){
            this.skillStatu.statu = 1;
            this.skillStatu.startTime = _statu[0].startTime;
            this.skillStatu.endTime = _statu[0].endTime;
            this.skillStatu.currentTime =_statu[0].currentTime;
            // this.skillStatu.currentTime = '2018-05-17 17:06:00';
            // this.skillStatu.startTime = '2018-05-17 17:06:05';
            // this.skillStatu.endTime = '2018-05-17 17:06:50';
            if(this.isCoundownCall){
              this.$refs.countDown.run({acEndTime:this.skillStatu.endTime,currTime:this.skillStatu.currentTime});
            }
          }else{ 
            _statu = this.skillProduct.filter((data)=>{
              return data.status == 4;//已结束
            });
            if(_statu.length){
              this.skillStatu.statu = 4;
            }
          }
        }
        //循环是为了设置时间
        for(var i=0;i<this.skillProduct.length;i++){
          var item = this.skillProduct[i];
          if(item.status==1){
            var curDate = item.startTime !=''? item.startTime.split(' ')[1].split(':') : '';
            var _m =curDate !=''?Number.parseInt(curDate[0]):'';
            var _d = curDate !=''?Number.parseInt(curDate[1]):'';
            this.skillProduct[i].curDate = _m+'.'+_d;
            // this.skillProduct[i].curDate = '6.5';
            console.log('this.skillProduct[i].curDate',this.skillProduct[i].curDate);
          }else{
            this.skillProduct[i].curDate = '';
          }
        }
      }
    },
    beforeDestroy() {

    }
}
</script>
