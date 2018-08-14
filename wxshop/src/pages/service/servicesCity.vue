<template>
	<section class="service">
    <article class="content s_contract serviceCity">
      <div class="s_city" v-for="item in cityList">
        <p class="c_p" :id="'#'+item.name">{{item.name}}</p>
        <ul>
          <li v-for="(city,index) in item.cities" :class="{'no_bor':index==item.cities.length-1}">{{city.city_name}}</li>
        </ul>
      </div>
      <ul class="letter">
        <li @click="goTOLetter(item)" v-for="item in letter">{{item}}</li>
      </ul>  
    </article>
    <div class="btn_letter" v-show="isShow">{{btnLetter}}</div>
  </section>
</template>
<script type="text/babel">
  import {setShopsId} from '../../utils/assist';
  import {codeKeyValue} from '../../utils/errorCode';
  import {cityData,letter,selfCity} from '../../utils/city';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        ok: false,
        tips1: false,
        isSingle: false,
        start1: false,
        title: '代驾服务',
        serviceCityList:[],
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/serve/rightsListDriveDot?update=true',
        isShow:false,
        btnLetter:'',
        cityList:[],
        letter:[]
      }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchCityData();
      this.cityList = selfCity.var;
      this.letter = letter;
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
      
      goTOLetter(letter){
        var that = this;
        this.btnLetter = letter;
        this.isShow = true;
        var d = document.getElementById('#'+letter);
        if(d){
          document.body.scrollTop = document.getElementById('#'+letter).offsetTop;
        }
        setTimeout(function(){
          that.isShow =false;
        },2000);
      },
      fetchCityData: function() {
        //获取首页数据
        let that = this;
        // that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl)
          .then((response) => {
            const data = response.data;
            if(data.code == "S_OK") {
              if(data.var.length>0){
                this.setCity(data.var);
              }
            }else{
              // that.$dialog.notify({mes: codeKeyValue[data.code], icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          },(response) => {
            that.tips1 = true;
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              // that.$dialog.loading.close();
            },300);
          });
      },
      //判断数组中是否有已数据
      getLetter(str,len){
        var s = false;
        for(var i=0;i<len.length;i++){
          if(str==len[i]){
            s = true;
            break;
          }
        }
        if(s){
          return s;
        }
      },
      pushList(opts,letter){
        var bol = false;
        for(var i=0;i<this.cityList.length;i++){
          var item = this.cityList[i];
          if(item.name == letter){
            var len = this.cityList[i].cities.filter(function(data){
              return data.city_name== opts.city_name
            });
            if(len.length==0){
              this.cityList[i].cities.push(opts);
            }
            bol = true;
            break;
          }
          return bol;
        }
      },
      //设置城市排序
      setCity(updateCity){
        var cityData1 = cityData.var;
        var cityList1 = updateCity;
        var len = [];
        var letterList = [];
        for(var i=0;i<cityList1.length;i++){
          var c_name = cityList1[i].city_name;
          for(var j=0;j<cityData1.length;j++){
            var j_name = cityData1[j].name;
            var isHas = false;
            for(var k=0;k<cityData1[j].cities.length;k++){
              if(cityData1[j].cities[k].name.indexOf(c_name)>=0){
                if(this.pushList(cityList1[i],j_name)){
                  isHas = true;
                  break;
                }
              }
            }
            if(isHas){
                break;
            }
          }  
        }
      }
    }
  }
</script>