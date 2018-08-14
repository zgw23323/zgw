<template>
	<transition name="page">
		<section class="ck-doc addressAdd">
      <div class="modify-bd">
        <yd-cell-group>
          <yd-cell-item>
              <span slot="left">收货人：</span>
              <input v-model="optsAddressData.consignee" slot="right" type="text" placeholder="请输入姓名" maxlength="12">
          </yd-cell-item>
          <yd-cell-item>
              <span slot="left">联系电话：</span>
              <input v-model="optsAddressData.phone" slot="right" type="text" placeholder="请输入手机号码" maxlength="11">
          </yd-cell-item>
            <yd-cell-item arrow>
                <span slot="left">选择区域：</span>
                <input slot="right" :ready="start1" type="text" @click.stop="show2 = true" v-model="model2" readonly class="cell-input" placeholder="请选择收货地址">
            </yd-cell-item>
            <yd-cell-item class="cell-position">
              <span slot="left">详细地址：</span>
              <input v-model="optsAddressData.address" slot="right" type="text" placeholder="请输入详细收货地址" maxlength="50">
              <i slot="i" class="i-position" @click="getLocation"></i>
          </yd-cell-item>
          <yd-cell-item type="checkbox">
              <div slot="left">设为默认</div>
              <input slot="right" type="checkbox" v-model="optsAddressData.defaultBoolean"/>
              <!--<yd-switch slot="right" v-model="optsAddressData.defaultBoolean"></yd-switch>-->
          </yd-cell-item>
        </yd-cell-group>
      </div>

      <yd-cityselect :ready="start1" :class="{'transparent': isTransparent == true ? true : false}"
                     v-model="show2"
                     :done="selectCityResult"
                     :provance="optsAddressData.provinceId"
                     :city="optsAddressData.cityId"
                     :area="optsAddressData.areaId"
      ></yd-cityselect>

      <div class="savebtn" @click="submit">保存</div>
		</section>
	</transition>
</template>

<script type="text/babel">
import {MP} from './../modules/map/index';
import {setShopsId} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
export default {
    data() {
        return {
        	title: '收货地址',
        	isTransparent: true,
          start1: false,
          show2: false,
          model2: '',
          apiAddressUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxAddShipperAddr?orgCode='+window.orgCode,
        	optsAddressData: {
        		consignee: '',
        		phone: '',
        		provinceId: '',
        		cityId: '',
        		areaId: '',
        		address: '',
        		defaultBoolean: false
        	},
        	apiGetAddressUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxFindShipperAddr?orgCode='+window.orgCode,
        }
    },
    watch: {
    	optsAddressData: {  //深度监听地址数据
	        handler() {
	        },
	        deep: true
	    },
	    '$route.query.type': 'markInit'  //监听当前路由变化（辅助初始化）
    },
    mounted() {
      //挂载完毕初始化页面数据
      this.markInit();
      //获取路由type表示切换对应功能页面
      if(this.$route.query.type == "update") {
        //修改地址获取数据
      	this.getAddress();
      }else {
        //新增地址初始自动获取当前位置
        this.getLocation();
      }
      //加个倒计时显示解决联动列表的闪屏问题
      setTimeout(()=>{
      	this.isTransparent = false;
      },1000);
  	},
    methods: {
    	  markInit() {
          //初始化页面标题
    		  document.title = this.title;
    		  //获取shopsid缓存数据
          setShopsId();
          this.$store.dispatch('log',{account:LOGCODE.HOME.PAGE_ADDRESSADD})
		    },
        isEmptyValid(value,allowEmptyString) {  //是否为空值校验
          let val= value.replace(/\s/g,"");
          return (val === null) || (val === undefined)
            || (!allowEmptyString ? val === '' : false)
            || (val.length === 0);
        },
        strlenValid(str) { //字符长度校验，中文2位，非中文1位返回
          var len = 0;
          for (var i = 0; i < str.length; i++) {
            var c = str.charCodeAt(i);
            if ((c >= 0x0001 && c <= 0x007e) || (0xff60<=c && c<=0xff9f)) {
              len++;
            }
            else {
              len += 2;
            }
          }
          return len;
        },
        selectCityResult(ret) {  //地址联动结果返回
        	this.optsAddressData.provinceId = ret.provance;
        	this.optsAddressData.cityId = ret.city;
        	this.optsAddressData.areaId = ret.area;
            this.model2 = ret.provance + ' ' + ret.city + ' ' + ret.area;
        },
        submit() { //提交结果数据
        	let that = this;
          //名字校验
          if(that.isEmptyValid(that.optsAddressData.consignee)) {
            that.$dialog.notify({mes: '请输入您的姓名', icon: 'error', timeout: 1200});
            return
          }
          if(that.strlenValid(that.optsAddressData.consignee) > 12) {
            that.$dialog.notify({mes: '输入名字过长', icon: 'error', timeout: 1200});
            return;
          }
          //手机号码校验
          if(that.isEmptyValid(that.optsAddressData.phone)) {
            that.$dialog.notify({mes: '请输入您的手机号码', icon: 'error', timeout: 1200});
            return;
          }
        	if(!/^[0-9]{11}\d*$/.test(that.optsAddressData.phone)) {
        		this.$dialog.notify({mes: '手机号码输入有误', icon: 'error', timeout: 1200});
        		return;
        	}
          //选择区域校验
          if(that.isEmptyValid(that.model2)) {
            this.$dialog.notify({mes: '请选择区域', icon: 'error', timeout: 1200});
            return;
          }
          //详细地址校验
          if(that.isEmptyValid(that.optsAddressData.address)) {
            this.$dialog.notify({mes: '请输入详细地址', icon: 'error', timeout: 1200});
            return;
          }
          if(that.strlenValid(that.optsAddressData.address) > 50) {
            that.$dialog.notify({mes: '输入详细地址过长', icon: 'error', timeout: 1200});
            return;
          }

          //提交数据
        	let apiType = this.$route.query.type;
        	if(apiType == "save") {
        		var apiUrl = that.apiAddressUrl+'&type=save&shopsid='+window.shopsid;
        	}else if(apiType == "update") {
        		var apiUrl = that.apiAddressUrl+'&type=update&shopsid='+window.shopsid;
        		that.optsAddressData.id = this.$route.query.id;
        	}
        	that.$dialog.loading.open('保存中...');
          that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_ADDRESSADD_SAVE});
	        that.$http.post(apiUrl,that.optsAddressData)
	          .then((response) => {
	            const data = response.data;
	            if(data.code == "S_OK") {
	            	setTimeout(()=>{
		              that.$router.back();
		            },300);
                that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_ADDRESSADD_SAVE_OK});
	            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                  that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                    that.$router.push('/login');
                  }});
                }else if(data.code == "NO_LOGIN") {
                  that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                    that.$router.push('/login');
                  }});
                }else {
	              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
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
        getAddress() {
          //修改地址，初始化请求地址数据
        	let that = this;
        	let opts = {};
        	opts.id = this.$route.query.id;
          this.$dialog.loading.open('加载数据中...');
        	that.$http.post(that.apiGetAddressUrl+'&shopsid='+window.shopsid,opts)
	          .then((response) => {
	            const data = response.data;
	            if(data.code == "S_OK") {
	            	that.optsAddressData = data.var;
			      	  that.model2 = that.optsAddressData.provinceId + ' ' + that.optsAddressData.cityId + ' ' + that.optsAddressData.areaId;
                that.start1 = true;
	            }else if(data.code == "LOGIN_SID_FAL_0X001" || data.code == "LOGIN_FAL_0x003" || data.code == "LOGIN_FAL_0x004" || data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '登录失效，请重新登录', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else if(data.code == "NO_LOGIN") {
                that.$dialog.notify({mes: '您还未登陆', icon: 'error', timeout: 1200, callback: () => {
                  that.$router.push('/login');
                }});
              }else {
                that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
              }
              setTimeout(()=>{
                this.$dialog.loading.close();
              },300);
	          },(response) => {
	            that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
	          });
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
        showPosition(position) {
          //百度地图通过坐标转换具体位置
          var that = this;
          var lat = position.coords.latitude;
          var lon = position.coords.longitude;

          MP().then(BMap => {
            var point = new BMap.Point(lon, lat);
            var gc = new BMap.Geocoder();
            gc.getLocation(point, function(rs) {
              document.title = that.title;
              var addComp = rs.addressComponents;
              that.optsAddressData.provinceId = addComp.province;
              that.optsAddressData.cityId = addComp.city;
              that.optsAddressData.areaId = addComp.district;
              that.optsAddressData.address = addComp.street;
              that.model2 = addComp.province + ' ' + addComp.city + ' ' + addComp.district;
              that.start1 = true;
            });
          });
        }
          
    }
}
</script>

