<template>
    <section class="service">
      <div class="select-accordion">
       <div class="filterbar-wrap" v-if="!tips1">
          <div class="filter-bar">
            <ul class="sort-tab">
              <li class="sort" @click="dropList(1)" :class="{'current selected': dropList1}">选择省<i class="arrow"></i></li>
              <li class="sort" @click="dropList(2)" :class="{'current selected': dropList2}">选择市<i class="arrow"></i></li>
              <li class="sort common-sort" @click="dropList(3)" :class="{'current selected': dropList3}"><span>医院性质</span><i class="arrow"></i></li>
            </ul>

            <div id="drop-list1" class="drop-list" v-if="dropList1">
               <ul class="drop-list-first">
                <li :class="'-1' == m1curId ? 'current' : ''" @click="fetchProvince('-1',1)">
                  <p>全部<i class="iconfont icon_left_only" v-if="m1curId=='-1'">&#xe72e;</i></p>
                </li>
                <li :class="province.id == m1curId ? 'current' : ''" v-for="(province,index) in provinceList" @click="fetchProvince(province.id,1)">
                  <p>{{province.name}}<i class="iconfont icon_left_only" v-if="m1curId==province.id">&#xe72e;</i></p>
                </li>
              </ul>
            </div>

            <div class="drop-list" v-if="dropList2">
              <ul class="drop-list-middle" style="overflow: scroll;max-height: 7rem;">
                <li :class="'-1' == m2curId ? 'current selected' : ''" @click="fetchProvince('-1',2)">
                  <p>全部<i class="iconfont icon_left_only" v-if="m2curId=='-1'">&#xe72e;</i></p>
                </li>
                <li :class="city.id == m2curId ? 'current selected' : ''" v-for="(city,index) in cityList" @click="fetchProvince(city.id,2)">
                  <p>{{city.name}}<i class="iconfont" v-if="m2curId==city.id">&#xe72e;</i></p>
                </li>
              </ul>
            </div>

            <div class="drop-list" v-if="dropList3">
              <ul class="drop-list-middle" style="overflow: scroll;max-height: 7rem;">
                <li :class="'全部' == nature ? 'current selected' : ''" @click="fetchProvince('全部',3)">
                  <p>全部<i class="iconfont icon_left_only" v-if="nature=='全部'">&#xe72e;</i></p>
                </li>
                <li :class="nature == item.nature ? 'current selected' : ''" v-for="(item,index) in natureList" @click="fetchProvince(item.nature,3)">
                  <p>{{item.nature}}<i class="iconfont icon_left_only" v-if="nature==item.nature">&#xe72e;</i></p>
                </li>
              </ul>
            </div>
          </div>
        </div>
        <div class="filterbar-mark" v-if="mark" @click="closeDropList"></div>
      </div>
      <div v-infinite-scroll="loadList" infinite-scroll-disabled="disabled" infinite-scroll-distance="10">
        <div class="hospital-content">
          <ul>
            <li v-for="(item,index) in hospitalList" :class="{'no_bor':index==hospitalList.length-1}" v-if="item.type==3&&index<opts.curPage*20">
              <div>
                <p class="hospital_p1">{{item.name}}</p>
                <p class="hospital_p2" style="margin-bottom: 0;">{{item.level}}·{{item.nature}}</p>
                <!-- <div class="hospital_p3"><i class='h_phone'>&nbsp;</i>电话：{{item.phone}}</div> -->
                <!-- <div class="hospital_p3"><i class='h_address'>&nbsp;</i>地址：{{item.address}}</div> -->
              </div>
              <div class="hospital_yu" v-if="type=='2'" ><span @click="chekcedBtn(item,1)">我要预约</span></div>
              <div class="hospital_yu" v-if="type=='1'"><i :class="{'che-btn noCheck':item.name !=optsCheck.name,'che-btn isCheck':item.name ==optsCheck.name}" @click="chekcedBtn(item,2)">&nbsp;</i></div>
            </li>
          </ul>
        </div>
      </div>
      <div class="tipsPage" v-if="tips1">
        <div class="tipsBox">
          <div class="tipStatus" data-type="busy"></div>
          <p>亲，网络出问题啦~~</p>
          <div class="btns" @click="reLoad">重新加载</div>
        </div>
      </div>
    <!-- <div :class="{'d_call_phone dis':isKeyUp,'d_call_phone':!isKeyUp}" @click = "submit" v-if="type=='1'">
      <a href="javascript: void(0);">确认选择</a>
    </div> -->
    <div class="d_call_phone" @click = "submit" v-if="type=='1'">
      <a href="javascript: void(0);">确认选择</a>
    </div>
    </section>
</template>

<script type="text/babel">
import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
import {codeKeyValue} from '../../utils/errorCode';
import {LOGCODE} from '../../utils/logCode';
export default {
    data() {
        return {
          disabled: true,
          mark: false,
          tips1: false,
          type:'',
          doneTipShow: true,
          dropList1: false,
          dropList2: false,
          dropList3: false,
          // isKeyUp:false
          m1curId: '-1',
          m2curId: '-1',
          nature:'全部',//选择的医院性质
          title: '就医帮手',
          // title: '就医帮手-绿色通到',
          isChecked:false,
          apiUrl: window.baseUrl+window.appName+'/shopapi/func/wxrights/WxRightsGethospitaldotlist?sid='+window.shopsid,
          opts: {curPage: 1, pageSize: 6},
          optsCheck:{},//选择的医院信息
          provinceList:[],//省份列表
          cityList:[],//城市列表
          natureList:[
            {
              nature:'公立医院'
            },
            {
              nature: '私立医院'
            }
          ], 
          menuSubData: [],
          allHospitalData:[],//存放所有的数据
          allHospitalList:[],//所有的医院列表
          hospitalList:[]//医院列表
        }
    },
    watch: {
      '$route.path': 'markInit' //监听当前路由变化（辅助初始化）
    },
    beforeMount() {//fetchList('m3',menu.code,menu.name)
      //挂载完毕初始化页面数据
       this.markInit();
       this.fetchServiceDetail();
       document.getElementById('body').style.height="auto";
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

          //拿到医院列表数据
          var hospitalData = getSessionStore('hospitalListData',true);
          if(typeof(hospitalData) !='boolean') {
            this.allHospitalData = hospitalData;
          }
          this.type = this.$route.query.type;
          this.$store.dispatch('log', {account:LOGCODE.SERVICE.PAGE_HOSPITAL});
        },
        closeDropList() {
          //关闭下拉筛选列表
          setTimeout(()=>{
            this.dropList2 = false;
            this.dropList3 = false;
            this.dropList1 = false;
            this.mark = false;
          },200);
        },
        getHospital(){
          var len = [];
          for(var i=0;i<this.cityList.length;i++){//循环城市列表，
            var item = this.cityList[i];
            for(var j=0;j<this.allHospitalData.length;j++){//找到属于城市的医院
              var o = this.allHospitalData[j];
              if(o.parent_id == item.id && o.type==3){
                if(this.nature =='全部'){//判断性质是否全选，非全选多一层过滤
                  len.push(o);
                }else if(o.nature == this.nature){
                  len.push(o);
                }
              }
            }
          }
          console.log(len);
          this.allHospitalList = len;
        },
        fetchProvince(id,type){
          var that = this;
          var len = [];
          if(type==1){//选择的是省份
            this.m1curId = id;
            this.m2curId = '-1';//选择省份，重置城市选中项
            if(id=='-1'){//省份全选的情况
              this.cityList = this.allHospitalData.filter(function(data){
                return data.type==2;
              });
              this.allHospitalList = this.allHospitalData.filter(function(data){
                if(that.nature !='全部'){//如果勾选了性质，多一层过滤
                  return data.type == 3 && data.nature == that.nature;
                }else{
                  return data.type == 3;
                }
                
              });
            }else{//选择某一个省份
              that.cityList = this.allHospitalData.filter(function(data){
                return data.type==2 && data.parent_id == id;
              });
              this.getHospital();
            }
          }else if(type == 2){//选择的是城市
            this.m2curId = id;
            if(id =='-1'){//城市全选
                if(that.m1curId !='-1'){//省份非全选的情况，当前城市列表为该省份城市列表
                  this.getHospital();
                }else{//省份全选、城市全选，当前城市列表为所有城市列表
                  if(this.nature !='全部'){//性质不全选，多一层过滤
                    this.allHospitalList = this.allHospitalData.filter(function(data){
                      return that.nature==data.nature && data.type==3;
                    });
                  }else{//性质全选
                    this.allHospitalList = this.allHospitalData.filter(function(data){
                      return data.type==3;
                    });
                  }
                }
            }else{//城市不全选，找到匹配该城市id值的医院列表
              if(this.nature !='全部'){//性质不全选，多一层过滤
                this.allHospitalList = this.allHospitalData.filter(function(data){
                  return data.parent_id == id && data.type==3 && data.nature == that.nature;
                });
              }else{//性质全选
                this.allHospitalList = this.allHospitalData.filter(function(data){
                  return data.parent_id == id && data.type==3;
                });
              }
            } 
          }else{//选择的是性质
              that.nature = id;
              if(that.m2curId !='-1'){//城市不为全部，匹配对应城市医院
                this.allHospitalList = this.allHospitalData.filter(function(data){
                  if('全部' != id){
                    return data.parent_id == that.m2curId && data.type==3 && data.nature == id;
                  }else{
                    return data.parent_id == that.m2curId && data.type==3;
                  }
                });
              }else if(this.m1curId =='-1'){//城市为全部、省份为全部
                this.allHospitalList = this.allHospitalData.filter(function(data){
                  if('-全部' != id){
                    return data.type==3 && data.nature == id;
                  }else{
                    return data.type==3;
                  }
                });
              }else{//城市为全部，选择了某一省份，当前城市列表为该省份所属城市
                this.getHospital();
              }
          }
          this.setPage();
          this.closeDropList();
        },
        loadList() {
          //分页加载页面数据
          let that = this;
          if(!that.disabled) {
            that.opts.curPage++;
          }else {
            return
          }
          if(that.opts.curPage > that.opts.total) {
            /* 所有数据加载完毕 */
            that.disabled = true;
            that.opts.curPage =  that.opts.total;
            return;
          }
          this.setPage();
        },
        dropList(num) {
          //展示下拉列表
          if(num ==1){
            this.dropList1 = !this.dropList1;
            this.dropList2 = false;
            this.dropList3 = false;
          }else if(num==2){
            this.dropList1 = false;
            this.dropList2 = !this.dropList2;
            this.dropList3 = false;
          }else{
            this.dropList1 = false;
            this.dropList2 = false;
            this.dropList3 = !this.dropList3;
          }
          if(this.dropList1||this.dropList2||this.dropList3){
            this.mark = true;
          }else{
            this.mark = false;
          }
        },
        chekcedBtn(item,type){//根据医院的parent_id为线索，逐级找到对应的城市、省份id;
          this.optsCheck = item;
          this.optsCheck.cityId = item.parent_id;
          var len = this.allHospitalData.filter(function(data){
            return data.id == item.parent_id && data.type ==2;
          });
          this.optsCheck.provinceId = len[0].parent_id;
          if(type==1){
            this.submit();
          }
        },
 
        fetchServiceDetail: function() { 
          //获取首页数据
          let that = this;
          this.disabled=true;
          that.$dialog.loading.open('加载数据中...');
          that.$http.post(that.apiUrl,{})
            .then((response) => {
              const data = response.data;
              if(data.code == "S_OK") {
                
                 setSessionStore("hospitalListData",data.var);
                 this.allHospitalData = data.var;
              }
              if(this.allHospitalData.length>0){
                that.disabled = false;
                this.setData();
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
        //设置页数
        setPage(){
          var len = this.allHospitalList.length;
          this.opts.total = len%20>0?parseInt(len/20+1):parseInt(len/20);
          if(this.opts.curPage>1){
            this.hospitalList = this.allHospitalList.slice(0,parseInt(this.opts.curPage)*20);
          }else{
            this.hospitalList = this.allHospitalList.slice((parseInt(this.opts.curPage)-1)*20,parseInt(this.opts.curPage)*20);
          }
        },
        setData(){
          for(var i=0;i<this.allHospitalData.length;i++){
            var item = this.allHospitalData[i];
            var len;
            if(item.type==1){//省份
              this.provinceList.push(item);
            }else if(item.type==2){//城市
              this.cityList.push(item);
            }else{//医院
              this.allHospitalList.push(item);
            }
          }
          this.setPage();
        },
        //确认选择医院跳回到信息录入页面
        submit(){
          //缓存选择医院信息
          setSessionStore('hospitalInfo',this.optsCheck);
          this.$router.push('/medicalAssistantSubmit');
        }
    }
}
</script>
