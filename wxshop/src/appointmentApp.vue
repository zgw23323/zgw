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
    }
     
  }
</script>
