<template>
    <section class="c-doc appointmentStore">
      <article class="nav_top">
        <nav>
          <div class="item" @click="areaBtn"><span>地区</span><i class="dowm">&nbsp;</i></div>
          <div class="item" @click="carBtn"><span>品牌</span><i class="dowm">&nbsp;</i></div>
        </nav>
      </article>
      <article class="store_content">
        <div class="cur_address"><i class="i_address">&nbsp;</i><span>深圳市南山区科技园工业大厦</span></div>
        <ul>
          <li :class="{'cur':curOrgId==item.id}" v-for="item,index in storeList" @click="storeBtn(item)">
            <div class="d_1">
              <span>{{item.orgShortName}}<span class="default" v-if="index==0">推荐</span></span>
              <span class="s_1"><span v-if="item.distance"><&nbsp;</span>{{item.distance}}km</span>
            </div>

            <p>电话：{{item.mobilePhone}}</p>
            <p>地址：{{item.address}}</p>
          </li>
        </ul>
      </article>

      <div class="m-popup-mask" v-show="tips1||tips2" @click="closeTips">&nbsp;</div>
      <article class="nav_tip">
        <div class="area_tip" v-show="tips1">
          <ul class="left">
            <li v-for="item,index in provienceList" @click="provienceBtn(item)"><span>{{item.provienceName}}</span><span class="cur_left" v-if="item.provienceId == isProvienceId">&nbsp;</span></li>
          </ul>
          <ul class="right">
            <li v-for = "item,index in cityList" @click="cityBtn(item)">{{item.cityName}} <span class="cur_right" v-if="item.cityId==isCityId">&nbsp;</span></li>
          </ul>
        </div>

        <div class="brand_tip" v-show="tips2">
          <ul>
            <li v-for="item,index in carList" @click="brandsBtn(item)"><span>{{item.brandName}}</span><span class="cur_right" v-if="checkCarId==item.brandId">&nbsp;</span></li>
          </ul>
        </div>
      </article>
    </section>
</template>

<script>
import {MP} from './../../modules/map/index';
import {getStore,setSessionStore,getSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import {orgPoint} from '../../utils/orgPoint';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '预约门店',
        start1:false,
        tips1: false,
        tips2: false,
        isProvienceId:'',
        isCityId:'',
        provienceList:[],
        cityList:[],
        carList:[],
        checkCarId:'',
        newGeo:{},
        curPoint:{lng:113.941401,lat:22.54664},//当前坐标
        optsAddressData: {
          consignee: '',
          phone: '',
          provinceId: '',
          cityId: '',
          areaId: '',
          address: '',
          defaultBoolean: false
        },
        curOrgId:'',
        newStore:[],
        storeList: []
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      document.title = this.title;
      this.getLocation();
      // this.getDistance({lng:"113.941317",lat:"22.546682"},{lng:"113.941772",lat:"22.546362"});
      ajaxModel.listOrgs(this,this.fecthCity);
      ajaxModel.wxListCarInfo(this,this.fecthCar);

      // this.searchByStationName('广州友道哈弗白云星级店','广州市');

      //获取缓存中门店
      var _orgOpt = getSessionStore('appointment_store',true);
      if(typeof(_orgOpt) != 'boolean'){
        this.curOrgId = _orgOpt.id;
      }
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_APPOINTMENTStore});
    },
    methods: {
      //地区菜单点击
      areaBtn() {
        this.tips1 = !this.tips1;
        this.tips2 = false;
      },
      //省份点击
      provienceBtn(obj){
        this.cityList = obj.cityList;
        this.isProvienceId = obj.provienceId;
        this.getProvienceOrgs(obj);
      },
      //城市点击
      cityBtn(obj) {
        this.tips1 = !this.tips1;
        this.isCityId = obj.cityId;
        this.storeList = obj.orgList.sort(this.compareDistance('distance'));
      },
      carBtn() {
        this.tips2 = !this.tips2;
        this.tips1 = false;
      },
      //获取省份下的门店
      getProvienceOrgs(obj){
        var _orgList = [];
        this.isCityId = '';
        for(var i=0;i < obj.cityList.length;i++){
          _orgList = [..._orgList,...obj.cityList[i].orgList];
        }
        // console.log('_orgList',_orgList);
        this.storeList = _orgList.sort(this.compareDistance('distance'));
      },
      //设置门店坐标
      setLocationOrgList(pArr){
        let that = this;
        var newOrgList = pArr;
        that.storeList = [];
        for(var i=0;i<newOrgList.length;i++){
          var item = newOrgList[i];
          var _point = {};
          var _pointOpt = orgPoint.filter((data)=>{
            return data.id == item.id;
          });
          newOrgList[i].distance = that.getDistance(that.curPoint,_pointOpt[0].point);
        }
        this.storeList = newOrgList.sort(this.compareDistance('distance'));
        // console.log('this.newOrgList',this.storeList);
      },
      compareDistance(property){
        return (a,b)=>{
          var value1 = a[property];
          var value2 = b[property];
           // console.log(value1,value2,'11111111');
          return value1 - value2;
        }
      },
      
      //品牌点击
      brandsBtn(opt){
        this.checkCarId = opt.brandId;
        this.getStoreList(this.checkCarId);
      },
      //获取有某个品牌的门店
      getStoreList(pId){
        var _storeList = [];
        for(var i=0;i<this.storeList.length;i++){
          var _itemList = this.storeList[i];
          var len = _itemList.list.filter((data)=>{
            return data.id == pId;
          });
          if(len.length>0){
            _storeList.push(_itemList);
          }
        }
        this.storeList = _storeList.sort(this.compareDistance('distance'));
      },
      //获取城市json
      fecthCity(result) {
        this.setLocationOrgList(result)
        // console.log('result',result);
        this.newStore = result;
        // this.test(result);//获取坐标测试方法
        this.provienceList =  ydFunLib.orgsCityList(result);
      },
      //获取品牌
      fecthCar(result) {
        this.carList = result.brands;
        // console.log('this.carList',this.carList );
      },
      getLocation() {
        //H5获取当前坐标
        if(navigator.geolocation) {
          document.title = '正在获取当前位置...';
          navigator.geolocation.getCurrentPosition(this.showPosition);
          setTimeout(()=>{
            document.title = this.title;
            this.start1 = true;
          },5000);
        }
      },
      closeTips() {
        this.tips1 = false;
        this.tips2 = false;
      },
      showPosition(position) {
          //百度地图通过坐标转换具体位置
          var that = this;
          var lat = position.coords.latitude;
          var lon = position.coords.longitude;
          MP().then(BMap => {
            var point = new BMap.Point(lon, lat);
            // console.log('point',point);
            var gc = new BMap.Geocoder();
            gc.getLocation(point, function(rs) {
              document.title = that.title;
              var addComp = rs.addressComponents;
              that.optsAddressData.provinceId = addComp.province;
              that.optsAddressData.cityId = addComp.city;
              that.optsAddressData.areaId = addComp.district;
              that.optsAddressData.address = addComp.province + addComp.city + addComp.district+addComp.street;
              that.model2 = addComp.province + ' ' + addComp.city + ' ' + addComp.district;
              that.start1 = true;
              that.curPoint.lat = rs.point.lat;
              that.curPoint.lng = rs.point.lng;
              that.setLocationOrgList(that.newStore);
              console.log('showPosition',that.curPoint);
            });
          });
      },
      test(result){
        for(var i=0;i<result.length;i++){
          var item = result[i];
          var _opt = orgPoint.filter((data)=>{
            return data.id == item.id;
          });
          // console.log('_opt',_opt[0].address,item.cityName,item.id);
          this.searchByStationName(_opt[0].address,item.cityName,item.id);
        }
      },
      //根据地址获取经纬度
      searchByStationName(addressKey,cityKey,id) {
        console.log('searchResult1',addressKey,cityKey);
        let that = this;
        if(navigator.geolocation){
          var map = new BMap.Geocoder();
          map.getPoint(addressKey, function(point){
          },cityKey);
        }
      },
      //获取两个坐标间的距离
      getDistance:function(pt1,pt2){
        // console.log('pt1 pt2',pt1,pt2);
        // 维度
        var lat1 = (Math.PI / 180) * pt1.lng;
        var lat2 = (Math.PI / 180) * pt2.lng;
        //纬度
        var lon1 = (Math.PI / 180) * pt1.lat;
        var lon2 = (Math.PI / 180) * pt2.lat;
         
        var R = 6371;
        // 两点间距离 km，如果想要米的话，结果*1000就可以了
        var d = Math.acos(Math.sin(lat1) * Math.sin(lat2) + Math.cos(lat1) * Math.cos(lat2) * Math.cos(lon2 - lon1)) * R;
        // console.log('distance',d*1000);
        return Number.parseFloat(d.toFixed(2));  
      },
      //门店点击
      storeBtn(pOpt){
        setSessionStore('appointment_store',pOpt);
        this.$router.push({path: '/appointmentService'});
      }
    }
  }
</script>
