<template>
  <yd-layout>
    <transition name="page">
      <section class="ck-doc myCar">
        <article class="car_list">
          <ul>
            <v-touch 
              tag="li" 
              @swipeleft="curShowDel(item.id)" 
              :key="item.id" 
              :swipe-options="{threshold: 20}" 
              @swiperight="curHideDel(item.id)" 
              :class="{'del': showDelvalidator(item.id) ? true : false,'curBtn':item.id==curId}"
              v-for="item,index in myCarList"
            >
              <div class="i_flex" @click="carBtn(item)">
                <div class="left"><img src="../../img/appointment/carLogo.png" /></div>
                <div class="right">
                  <p class="p1"><span>{{item.carsName}}</span><span class="default" v-if="item.isDefault==1">默认</span></p>
                  <p class="p2">车牌号：{{item.licensePlate}}</p>
                  <p class="p2" v-if="item.frameNumber">车架号：{{item.frameNumber}}</p>
                </div>
              </div>
              <div class="delete_btn i_flex">
                <div class="btn_default" @click="setDefault(item,index)">设为</br>默认</div>
                <div class="btn_del" @click="deleteCar(item)">删除</div>
              </div>
              <!-- <div class="btn_default" @click="deleteCar(item,index)">删除</div> -->
          </v-touch>
          </ul>
        </article>
        <div class="d_fixed" @click="goTo"><div class="fixed_btn add">+添加爱车</div></div>
      </section>
    </transition>
  </yd-layout>

</template>

<script>
import {getStore,setSessionStore,getSessionStore,removeSessionStore} from '../../utils/assist';
import {LOGCODE} from '../../utils/logCode';
import {ydFunLib} from '../../utils/ydFunLib';
import ajaxModel from '../../utils/ajaxModel';
  export default {
    data() {
      return {
        title: '我的车辆',
        curShowDelId: [],
        curId:'',
        myCarList:[]
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      //获取缓存数据
      this.$store.dispatch('log', {account:LOGCODE.MAKEANAPPOINTMENT.PAGE_APPOINTMENTSERVICE});
    },
    methods: {
      markInit() {
        //初始化页面标题
        document.title = this.title;
        let carList = getSessionStore('myCarList_info',true);
        if(typeof(carList) != 'boolean'){
          this.myCarList = carList;
          this.setCheckCar();
        }else{
          ajaxModel.wxMyCars(this,this.fetchCar);
        }
      },
      curShowDel(id) {
        //显示删除按钮
        // console.log('delete');
        if(!this.showDelvalidator(id)) {
          this.noRepeatAddArray(id, this.curShowDelId);
        }
      },
      curHideDel(id) {
        //隐藏删除按钮
        if(this.showDelvalidator(id)) {
          this.noRepeatAddArray(id, this.curShowDelId);
        }
      },
      setCheckCar(){
        var _carOpt = getSessionStore('checkCar_info',true);
        if(typeof(_carOpt) !='boolean'){
          this.curId = _carOpt.id;
        }
      },
      showDelvalidator(value) {
        //是否显示删除按钮id校验
        return this.curShowDelId.indexOf(value) > -1;
      },
      noRepeatAddArray(selected, selectArr) {
        //数组去重过滤方法
        let len = selectArr.length + 1;
        for(let i = 0; i < len; i++){
            if(selected == selectArr[i]) {
              selectArr.splice(i,1);
              return;
            }
        };
        selectArr.push(selected);
      },
      goTo() {
        this.$router.push({
          path: '/addCar',
          query: {}
        });
      },
      //删除车辆
      deleteCar(pOpt,pIndex){
        let that = this;
        // console.log('deleteId',pOpt.id);
        ajaxModel.wxDelUserCar(this,() =>{
          // that.myCarList.splice(pIndex,1);
          var opt = getSessionStore('checkCar_info',true);
          if(typeof(opt) != 'boolean' && pOpt.id == opt.id){
             removeSessionStore('checkCar_info');
          }
          ajaxModel.wxMyCars(that,that.fetchCar);
          // setSessionStore('myCarList_info',that.myCarList);
        },pOpt.id)
      },
      //设置默认车辆
      setDefault(pOpt,pIndex) {
        let that = this;
        ajaxModel.wxSetDefaultUserCar(this,() =>{
          that.myCarList[pIndex].isDefault = 1;
          for(var i=0;i<that.myCarList.length;i++){
            if(that.myCarList[i].id != pOpt.id){
              that.myCarList[i].isDefault = 2;
            }
          }
          setSessionStore('myCarList_info',that.myCarList);
        },pOpt.id)
      },
      carBtn(pOpt){
        setSessionStore('checkCar_info',pOpt);
        this.$router.push({
          path: '/appointmentService',
          query: {}
        });
      },
      fetchCar(result){
        this.myCarList = result;
        // console.log('myCarList',this.myCarList);
        setSessionStore('myCarList_info',this.myCarList);
        this.setCheckCar();
      }
    }
  }
</script>
