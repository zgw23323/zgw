<template>
	<section class="seachRecord">
    <article class="search_glass" id="item1">
        <div class="s_glass_content">
          <input type="text" v-model="optApply.searchKey" placeholder="请输入车牌号或车架号" class="inp_cart_number">
        </div>
        <p class="seach_mess">{{seachMess}}</p>
        <div class="g_seach_btn" @click="seachRecord">查询</div>
    </article>
    <article class="new_order" id="item2" v-if="applyList && applyList.length>0">
      <p id="p1">预约维修客户</p>
      <ul id="ul">
        <li v-for="item in applyList" @click="goToRouter(item)">
          <div class="li_item1">
            <div>车牌号/车架号：<span>{{item.frameNumber}}</span></div>
          </div>
          <div class="li_item3">姓名：<span>{{item.customerName}}</span></div>
          <div>预约时间：<span>{{item.appointedStartTime.substring(0,4)}}年{{item.appointedStartTime.substring(5,7)}}月{{item.appointedStartTime.substring(8,10)}}日 {{item.appointedStartTime.substring(11,16)}}-{{item.appointedEndTime.substring(11,16)}}</span></div>
          <div class="li_item2" v-if="item.flowStatus==-1">审核状态：<span class="s_fail">已驳回</span></div>
        </li>
      </ul>
    </article>
     
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        title: '玻璃保修服务',
        apiApplyList: window.baseUrl+window.appName+'/shopapi/func/insurance/wxQueryApplyList?openId='+window.openId,//获取预约记录列表
        apiSearchApplyUrl: window.baseUrl+window.appName+'/shopapi/func/insurance/wxSearchApplyList',//查询预约记录
        seachOpt:{},
        seachMess:' ',
        height:'',
        total:0,
        applyList:[],
        optSeach: {
          curPage:1,
          pageSize:8
        },
        optApply:{
          searchKey:''//查询的车牌号或车架号
        }
      }
    },
    watch: {
      '$route.path': 'markInit'//监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      // this.fetSelectData();
      this.fecthApplyData();
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
        
        //拿到缓存中最近搜索的内容
        var _searchKey = getSessionStore('seachRecord_key',false);
        if(typeof(_searchKey) != 'boolean'){
          this.optApply.searchKey = _searchKey;
        }
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_SEACHRECORD});
      },
      loadList() {
        //分页加载页面数据
        let that = this;
        that.optSeach.curPage++;
        if(that.optSeach.curPage > that.total) {
          /* 所有数据加载完毕 */
          that.optSeach.curPage = that.total;
          return;
        }
        that.$http.post(this.apiApplyList,this.optSeach)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              for(var i=0;i<data.var.recordList.length;i++) {
                that.applyList.push(data.var.recordList[i]);
              }
            }else {
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            /* 单次请求数据完毕 */
            //window.$yduiBus.$emit('ydui.infinitescroll.finishLoad');

          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          });
      }, 
       //设置滑动区域的最大高度
      setScrollHeight(){
        var bodyHeight = document.body.offsetHeight;
        var item1 = document.getElementById("item1").offsetHeight;
        var p1 = document.getElementById("p1").offsetHeight;
        var _height = bodyHeight-item1-p1-10;//10像素是外边距的距离
        this.height = _height;
        $('ul').attr('style','max-height:'+_height+'px');
        // document.getElementById('ul').style='max-height: '+_height+'px';
        document.getElementById('ul').addEventListener('scroll', this.loadList);
      },
       
      goToRouter(opt) {
        this.$router.push({
          path: '/glassClaim',
          query:{'id':opt.id}
        });
      },
      fecthApplyData() {
         let that = this;
        this.$dialog.loading.open('加载中...');
        this.$http.post(this.apiApplyList,this.optSeach)
          .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            if(data.var.totalPage>0){
              that.total = data.var.totalPage;
              that.applyList = data.var.recordList;
              // that.setScrollHeight();
            }
            if(data.var.totalPage>0){
              setTimeout(()=>{
                that.setScrollHeight();
              },300);
            }
          }
        },(response) => {
          setTimeout(()=>{
            this.$dialog.loading.close();
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          },300);
        });
      },
      seachRecord(){
        let that = this;
        //提交数据
        if(that.optApply.searchKey == ''){
          that.$dialog.notify({mes: '请输入查询的车牌号或车架号', icon: 'error', timeout: 1200});
          return;
        } 
        setSessionStore('seachRecord_key',this.optApply.searchKey);
        this.$dialog.loading.open('加载中...');
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.BTN_PAGE_SEACHRECORD_SEARCH});
        this.$http.post(this.apiSearchApplyUrl,this.optApply)
          .then((response) => {
          const data = response.data;
          this.$dialog.loading.close();
          if(data.code == "S_OK") {
            that.seachMess = '';
            that.total = 1;
            that.applyList = data.var;
            that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_SEACHRECORD_SEARCH_OK});
            if(data.var !=null && data.var.length==1){
              that.goToRouter(data.var[0]);
            }else if(data.var !=null && data.var.length>1){
              setTimeout(()=>{
                that.setScrollHeight();
              },300);
            }
          }else {
            that.$store.dispatch('log', {account:LOGCODE.WORKERPASS.MESS_PAGE_SEACHRECORD_SEARCH_FAIL});
            that.applyList = [];
            if(data.code == 'NO_DATA'){
              that.seachMess = "没有找到申请维修记录";
            }else if(data.code == "PARAM_ISNULL"){
              that.seachMess = "请输入有效的车牌号或车架号";
            }else if(data.code == "NO_OPEN_SERVICE"){
              that.$dialog.notify({mes: '没有开通玻璃险索赔权限', icon: 'error', timeout: 1200});
            }else{
              that.$dialog.notify({mes: '查询失败', icon: 'error', timeout: 1200});
            }
          }
        },(response) => {
          setTimeout(()=>{
            this.$dialog.loading.close();
            this.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
          },300);
        });
      }
    }
  }
</script>