<template>
	<transition name="page">
		<section class="ck-doc address">
			<div class="address-add" @click="address('save')">+新建收获地址</div>
			<ul class="address-list">
				<li v-for="(list,index) in addressList">
					<div class="address-mes" @click="change(index, list.id)">
						<p class="p1" :class="{'address-default': list.isDefault == 1 ? true : false}"><span class="name">{{list.consignee}}</span>{{list.phone}}</p>
						<p class="p2">{{list.provinceId}}{{list.cityId}}{{list.areaId}}{{list.address}}</p>
					</div>
          <div class="address-bar cl">
            <div class="delete fl" @click="deleteAddress(list.id)"></div>
            <div class="edit fl" @click="address('update',list.id)"></div>
          </div>
				</li>
			</ul>
		</section>
	</transition>
</template>

<script type="text/babel">
import {setShopsId,setSessionStore} from '../utils/assist';
import {LOGCODE} from '../utils/logCode';
export default {
  data() {
    return {
      title: '收货地址',
      apiAddressUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxListShipperAddr?orgCode='+window.orgCode,
      addressList: [],
      apiDelAddressUrl: window.baseUrl+window.appName+'/shopapi/func/user/wxDelShipperAddr?orgCode='+window.orgCode
    }
  },
  watch: {
    '$route.path': 'markInit'   //监听当前路由变化（辅助初始化）
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
    this.fetchAddressList();
  },
  methods: {
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();

      this.$store.dispatch('log', {account:LOGCODE.HOME.PAGE_ADDRESS});
    },
    address(apiType,addressId) {
      //跳转到地址（修改/新增）
      this.$router.push({
        path: '/addressAdd',
        query: {
          type: apiType,
          id: addressId
        }
      });
    },
    change(i, id) {
      //切换地址
      if(this.$route.query.mark == 'change') {
         setSessionStore("_addressIndex", i);
        setSessionStore("_addressIndex_id", id);
        this.$router.back();
      }
    },
    deleteAddress(addressID) {
      //删除地址请求方法
      let that = this;
      let optsDelete = {};
      optsDelete.id = addressID;

      that.$dialog.loading.open('删除中...');
      that.$store.dispatch('log', {account:LOGCODE.HOME.BTN_PAGE_ADDRESS_DELETE});
      that.$http.post(that.apiDelAddressUrl+'&shopsid='+window.shopsid,optsDelete)
        .then((response) => {
          const data = response.data;
          if(data.code == "S_OK") {
            //重新加载地址列表
            that.fetchAddressList();
            that.$store.dispatch('log', {account:LOGCODE.HOME.MESS_PAGE_ADDRESS_DELETE_OK});
            that.$dialog.notify({mes: '删除地址成功', icon: 'error', timeout: 1200});
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
          setTimeout(()=>{
            that.$dialog.loading.close();
          },300);
          that.$dialog.notify({mes: '网络异常，请稍后再试！', icon: 'error', timeout: 1200});
        });
    },
    fetchAddressList() {
      //获取地址列表请求方法
      let that = this;
      that.$dialog.loading.open('加载数据中...');
      that.$http.post(that.apiAddressUrl+'&shopsid='+window.shopsid)
        .then((response) => {
          const data = response.data;
          if(data.code == "S_OK") {
            that.addressList = data.var;
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
    }
  }
}
</script>
