<template>
	<section class="service">
    <article class="content s_contract v_roomDetail">
      <div class="v_city" v-for="list in cityList">
        <p class="c_p">{{list.areaName}}</p>
          <div :class="{'item':index<list.networks.length,'item no_bor':index==list.networks.length-1}" @click = "cityDetail(list.areaName,item.stationName)" v-for="(item,index) in list.networks">
            <span>{{item.stationName}}</span>
          </div>
      </div>
    </article>
    <yd-bottom-tip v-if="tips2">
        <div class="item1">
          <div class="left">
            <img v-if="optAction.isHasImg" :src="imgBaseUrl+optAction.pics[0].picId" />
            <div class="cen">
              <div>{{optAction.stationName}}<span>{{optAction.stationHallType}}</span></div>
              <div class="f_36">{{optAction.stationHall}}</div>
            </div>
          
          </div>
          <div class='s_close' @click='tipsCancel(2)'>&nbsp;</div> 
        </div>
        <div class="get_btn" @click='receiveService'>我要领取</div>
        <div class="item2">
          <div class="s_flex bor"><div><span>营业时间：</span>{{optAction.stationHallTime}}</div></div>
          <div class="s_flex bor">
              <div>
                <span>服务电话：</span>{{optAction.tel}}
              </div>
              <i class="s_phone">&nbsp;</i>
            </div>
          <div class="s_flex">
            <div class="s_flex_add">
              <span>网点地址：</span>
              <div class="add_mess">{{optAction.position}}</div>
            </div>
            <i class="s_tip_address">&nbsp;</i>
          </div>
          <div :class="{'b_img':!optAction.isHasImg}">
            <img v-if="optAction.isHasImg" :src="imgBaseUrl+optAction.pics[1].picId" />
          </div>
        </div>
    </yd-bottom-tip>
    <div class="ck-pop-toast" v-if="tips1">
      <div class="inner">
        <div class="ck-pop">
          <div class="ck-pop-bd">
            <p class="g_succes">恭喜您，领取成功！</p>
            <ul>
              <li>服务：{{optReceive.serveName}}</li>
              <li>卡号：{{optReceive.mc_no}}</li>
              <li>密码：{{optReceive.mc_pwd}}</li>
            </ul>
            <div class="s_tip_submit" @click = "tipsCancel(1)">确定</div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        tips1: false,
        tips2: false,
        title: '高铁贵宾厅服务网点',
        imgBaseUrl: window.imgBaseUrl,
        apiVipRoomCityUrl: window.baseUrl+window.appName+'/shopapi/func/serve/rightsListSpeedDot',//高铁贵宾厅服务城市
        apiCityDetail:window.baseUrl+window.appName+'/shopapi/func/serve/rightsFindNetwork?shopsid='+window.shopsid,//城市详情
        apiReceiveServe:window.baseUrl+window.appName+'/shopapi/func/serve/rightsReceiveSpeed?shopsid='+window.shopsid,//领取
        cityList:[],
        optReceive:{},//领取成功后返回信息
        optAction: {}//活动信息对象
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchServiceCityData();
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
        this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_VIPROOMDETAIL});
      },
      cityDetail(str1,str2){
        let that = this;
        var areaName = that.cityList.filter(function(data){
          return data.areaName == str1;
        });
        console.log(areaName);
        if(areaName.length>0){
          var stationName = areaName[0].networks.filter(function(data){
              return data.stationName == str2;
          });
          that.optAction = stationName[0];
          if(that.optAction.pics.length>0){
            that.optAction.isHasImg = true;
          }else{
            that.optAction.isHasImg = false;
          }
          that.openTip(2);
        }
      },
      openTip(tips){
        this['tips'+tips] = true;
      }, 
      tipsCancel (tips){
        this['tips'+tips] = false;
      },
      //领取服务
      receiveService () {
        var that = this;
        that.$http.post(that.apiReceiveServe)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.optReceive = data.var;
              that.openTip(1);
              that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_VIPROOMDETAILADD_OK});
            }else{
              that.$store.dispatch('log', {account:LOGCODE.SERVICE.MESS_PAGE_VIPROOMDETAILADD_FAIL});
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
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
      fetchServiceCityData: function() {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiVipRoomCityUrl)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              that.cityList = data.var;
            }else{
              that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
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