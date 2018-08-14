<template>
  <router-view></router-view>
</template>

<script type="text/babel">
  // import {setConfig} from './utils/config';
  export default {
    data() {
      return {
        ok: false,
        start1: false,
        title: '',
        imgBaseUrl: window.imgBaseUrl,
        apiUrl: window.baseUrl+window.appName+'/shopapi/func/config/config?orgCode='+window.orgCode,
        opts: {curPage: 1, pageSize: 8, isBanner: 1}
      }
    },
    mounted() {
      //挂载完毕初始化页面数据
      if(window.orgCode != null && window.orgCode != '' && window.orgCode != undefined) {
        // this.fetchData();
      }
      
    },
    methods: {
      fetchData() {
        //获取首页数据
        let that = this;

        that.$dialog.loading.open('加载数据中...');
        that.$http.options[credentials]=true;
        that.$http.post(that.apiUrl,that.opts,{credentials:true})
          .then((response) => {
            const data = response.data;
            console.log("配置信息:", data);
            that.tips1 = false;
            if(data.code == "S_OK") {              
              var configOrgData = data.var;
              var orgInfo = configOrgData.orgInfo;
              if(orgInfo != null && orgInfo != undefined) {
                // var orgInfo_name = orgInfo.orgName;
                // var orgInfo_telPhone = orgInfo.phone;
                // var orgInfo_address = orgInfo.address;

                //更新门店地址信息
                // window.shop = {
                //     name: orgInfo_name,
                //     telphone: orgInfo_telPhone,
                //     address: orgInfo_address
                // }
                // setConfig(configOrgData);
              }
            }else if(data.code == "JSON_FAL") {             
              that.$dialog.notify({mes: '参数解析异常', icon: 'error', timeout: 1200});            
            }else {              
              that.$dialog.notify({mes: data.code, icon: 'error', timeout: 1200});
            }
            setTimeout(()=>{
              that.$dialog.loading.close();
            }, 300);
          },(response) => {            
            that.$dialog.notify({mes: '服务器请求异常', icon: 'error', timeout: 1200});
            setTimeout(()=>{
              that.$dialog.loading.close();
            },300);
          });
      }
    }
  }
</script>
