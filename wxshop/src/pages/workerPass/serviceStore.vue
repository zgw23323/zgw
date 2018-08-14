<template>
  <section class="service serviceStore">
    <article>
      <div class="store_li" v-for="item in storeList">
        <p class="c_p" :id="'#'+item.provienceId">{{item.provienceName}}省</p>
        <ul>
          <li v-for="(store,index) in item.storeList" :class="{'no_bor':index==item.storeList.length-1}" :id="store.id" @click="storeBtn(store)">
              <div class="s_flex">
                <div class="s_store_name">{{store.orgShortName}}</div>
                <div :class="{'s_store_logo s_store_cur':store.id==btnStore,'s_store_logo s_store_no':store.id!=btnStore}">&nbsp;</div>
              </div>
              <div class="s_flex"><!-- <i class="icon icon_2">&nbsp;</i> --><span>电话：</span><span>{{store.mobilePhone}}</span></div>
              <div class="s_flex"><!-- <i class="icon icon_3">&nbsp;</i> --><div class="s_flex1"><span>地址：</span><span>{{store.address}}</span></div></div>
          </li>
        </ul>
      </div>
     <!--  <ul class="letter">
        <li @click="goTOLetter(item)" :class="{'s_cur':btnLetter==item.provienceName}" v-for="item in storeList">{{item.provienceName}}</li>
      </ul>  --> 
    </article>
    <!-- <div class="btn_letter" v-show="isShow">{{btnLetter}}</div> -->
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
  import {LOGCODE} from '../../utils/logCode';
  import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        tips1: false,
        title: '选择4S店',
        isShow:false,
        btnLetter:'',
        btnStore:'',
        storeList:[]
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      ajaxModel.listOrgs(this,this.fetchStoreList);
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
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_SERVICESTORE});
        document.body.scrollTop = document.documentElement.scrollTop = 0;

        var storeOpt = getSessionStore('serviceStore_btn_info',true);
        if(typeof(storeOpt) !='boolean'){
          this.btnStore = storeOpt.id;
        }
      },
      
      goTOLetter(opt){
        var that = this;
        // this.btnLetter = opt.provienceName;
        // this.isShow = true;
        // var d = document.getElementById('#'+opt.provienceId);
        var d = document.getElementById(this.btnStore);
        if(d){
          // document.body.scrollTop = document.getElementById('#'+opt.provienceId).offsetTop;
          document.body.scrollTop = document.getElementById(this.btnStore).offsetTop-100;
        }
        // setTimeout(function(){
        //   that.isShow =false;
        // },2000);
      },
      fetchStoreList(result) {
        //获取地址列表请求方法
        let that = this;
        for(var i=0;i<result.length;i++){ 
          var _i = result[i];
          var len = that.storeList.filter(function(data){
            return data.provienceId == _i.provienceId;
          });
          if(len ==0){
            var item = {};
            item.provienceName = _i.provienceId == '30164'?"广西":_i.provienceName.split('省')[0];
            item.provienceId = _i.provienceId;
            item.storeList = [];
            item.storeList.push(_i);
            that.storeList.push(item);
          }else{
            for(var j=0;j<that.storeList.length;j++){
              if(that.storeList[j].provienceId == _i.provienceId){
                that.storeList[j].storeList.push(_i);
              }
            }
          }
        }
      },
      //门店点击
      storeBtn(opt){
        this.btnStore = opt.id;
        setSessionStore('serviceStore_btn_info',opt);
        if(this.$route.query.from=='glassSupportEdit'){
          this.$router.push({
            path: '/'+this.$route.query.from,
            query: {
              type: '2',
              id:this.$route.query.id,
              code:this.$route.query.code
            }
          });
        }else{
          this.$router.push({
            path: '/'+this.$route.query.from,
            query: {code:this.$route.query.code}

          });
        }
      }
    }
  }
</script>