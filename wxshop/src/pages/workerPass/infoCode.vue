<template>
  <transition name="page">
    <section class="infoCode">
        <article class="code_body">
          <div class="con">
            <!-- <div class="code_img" :style="{backgroundImage: 'url(' + codeUrl + ')'}"> -->
              <!-- <i class="i_logo">&nbsp;</i> -->
            <!-- </div> -->
            <img :src="codeUrl" class="code_img" />
            <p class="p1" v-if="refereeOpt.realName">业务员：{{refereeOpt.realName}}</p>
            <p class="p2">长按识别图中二维码会员充值更方面噢</p>
          </div>
        </article>
    </section>
  </transition>
</template>

<script type="text/babel">
import {setShopsId,getSessionStore,getStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import ajaxModel from '../../utils/ajaxModel';
export default {
    data() {
        return {
          title: '二维码',
          imgBaseUrl: window.imgBaseUrl,
          refereeOpt:{
            id:'',
            realName:''
          },
          codeUrl:'http://192.168.8.118:8082/shop/downLoad/showLogo/7f0000015f7015a1815f9002eb7600d1?type=thumb'
        }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        //获取shopsid缓存数据
        setShopsId();
        //获取缓存个人数据
        var _url = getSessionStore('code_info_url');
        if(typeof(_url) != 'boolean'){
          this.codeUrl = _url;
        }
        //从缓存中获取推荐人id
        var _refereeOpt = getStore('infoCode_refereeOpt',true);
        if(typeof(_refereeOpt) != 'boolean'){
          this.refereeOpt.id = _refereeOpt.id;
          this.refereeOpt.realName = _refereeOpt.realName;
        }else {
          var _id = this.codeUrl.split('refereeId=');//从二维码链接上拿推荐人ID
          var _personData = getSessionStore("workerInfo",true);
          if(typeof(_personData) !='boolean'&& _id.length>1){
            ajaxModel.wxListEmployee(this,this.setReferee,_personData.orgId);
          }
        }
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_INFOCODE});
      },
      setReferee(result){
        let that = this;
        var _id = this.codeUrl.split('refereeId=')[1];//从二维码链接上拿推荐人ID
        var len = result.filter(function(data){
          return data.id == _id;
        });
        if(len.length){
          this.refereeOpt.id = len[0].id;
          this.refereeOpt.realName = len[0].realName;
        }
      },
      tipOpen() {
        this.tips1 = !this.tips1;
      }
    }
}
</script>
