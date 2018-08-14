<template>
	<section class="service serviceCart">
    <article class="content s_contract serviceCity">
      <div v-if="carsName !=''" class="car_fixed_mess">当前选择的车系是<span>{{carsName}}</span></div>
      <div :class="{'cart_seach':carsName =='','cart_seach c_padd':carsName !=''}">
        <div>
          <i class="iconfont">&#xe602;</i>
          <input type="text" v-model="searchKey" />
        </div>
      </div>
      <div class="s_city" v-for="item in carData" v-if="item.carsList && item.carsList.length>0">
        <p class="c_p" :id="'#'+item.name">{{item.brandName}}</p>
        <ul>
          <li :id="car.carsId" v-for="(car,index) in item.carsList" :class="{'no_bor flex_between':index==item.carsList.length-1,'flex_between':index!=item.carsList.length-1}" @click="carBtn(car)">{{car.carsName}} <i class="car_gou" v-if="carsId == car.carsId">&nbsp;</i></li>
        </ul>
      </div>
    </article>
  </section>
</template>
<script type="text/babel">
  import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
  import { searchFromData, trim } from '../../modules/filters';
  import {codeKeyValue} from '../../utils/errorCode';
  import {cityData,letter,selfCity} from '../../utils/city';
  import {LOGCODE} from '../../utils/logCode';
  export default {
    data() {
      return {
        title: '玻璃保障服务',
        serviceCityList:[],
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/glassposition/wxListCarInfo',
        isShow:false,
        btnLetter:'',
        searchKey:'',
        cityList:[],
        carsId:'',
        carsName:'123',
        searchFlag:true,
        carData: [],
        letter:[]
      }
    },
    watch: {
      '$route.path': 'markInit', //监听当前路由变化（辅助初始化）
      searchKey() { //实时联动搜索关键词
        let _searchKey = trim(this.searchKey,'g');
        if(_searchKey != "") {
          this.carsName='',
          this.carsId = '',
          this.searchFlag = false;
          setTimeout(()=>{
            this.searchFlag = true;
            var cartList = getSessionStore('cart_list_data',true);
            if(typeof(cartList) !='boolean'){
              this.carData = [];
              for(var i=0;i<cartList.length;i++){
                var filtersDatas = [];
                if(cartList[i].carsList.length>0){
                  filtersDatas = searchFromData(_searchKey, cartList[i].carsList, 'carsName');
                  console.log('filtersDatas',filtersDatas);
                }
                if(filtersDatas.length>0){
                  this.carData.push(cartList[i]);
                  this.carData[this.carData.length-1].carsList = filtersDatas;
                } 
              }
            }else{
              this.fetchCityData(_searchKey);
            }
          },1000);
        }else {
          this.fetchCityData();
        }
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      this.fetchCityData();
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
        var carOpt = getSessionStore('carName',true);
        if(typeof(carOpt) !='boolean'){
          this.carsId = carOpt.carsId;
          this.carsName = carOpt.carsName;
        }
        this.$store.dispatch('log', {account:LOGCODE.WORKERPASS.PAGE_CARLIST});
        document.body.scrollTop = document.documentElement.scrollTop = 0;
      },
      
      goTOLetter(opt){
        var that = this;
        // this.btnLetter = opt.provienceName;
        // this.isShow = true;
        // var d = document.getElementById('#'+opt.provienceId);
        var d = document.getElementById(this.carsId);
        if(d){
          // document.body.scrollTop = document.getElementById('#'+opt.provienceId).offsetTop;
          document.body.scrollTop = document.getElementById(this.carsId).offsetTop-50;
        }
       
      },
      fetchCityData: function(mess) {
        //获取首页数据
        let that = this;
        that.$dialog.loading.open('加载数据中...');
        that.$http.post(that.apiUrl)
          .then((response) => {
            const data = response.data;
            that.$dialog.loading.close();
            if(data.code == "S_OK") {
              if(data.var.brands.length>0){
                setSessionStore('cart_list_data',data.var.brands);
                var cartList = data.var.brands;
                console.log('mess',mess);
                if(mess){
                  that.carData = [];
                  for(var i=0;i<cartList.length;i++){
                    var filtersDatas = [];
                    if(cartList[i].carsList.length>0){
                      filtersDatas = searchFromData(mess, cartList[i].carsList, 'carsName');
                      console.log('filtersDatas',filtersDatas);
                    }
                    if(filtersDatas.length>0){
                      that.carData.push(cartList[i]);
                      that.carData[that.carData.length-1].carsList = filtersDatas;
                    } 
                  }
                }else{
                  that.carData = data.var.brands;
                }
              }
            } 
            setTimeout(()=>{
              // that.goTOLetter();
            },20);
          },(response) => {
            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      },
      carBtn(opt){
        this.carsName = opt.carsName;
        setSessionStore('carName',opt);
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