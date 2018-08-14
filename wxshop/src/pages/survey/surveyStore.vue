<template>
  <section class="surveyStore">
     <div class="bg">
        <p class="prompt">请选择您所在门店:</p>
      
        <div class="kuai">
          <button class="option" v-for="val in Store" :class="{select: StoreId == val.id}" v-on:click="storeBtn(val.id)">{{val.name}}</button>
        </div>
    </div>
  </section>
</div>
</template>

<script type="text/babel">
import { mapState } from 'vuex';
import {setShopsId,setSessionStore,getSessionStore} from '../../utils/assist';
import {codeKeyValue} from '../../utils/errorCode';
import {LOGCODE} from '../../utils/logCode';
import ajaxModel from '../../utils/ajaxModel';
export default { 
  data() {
    return {
        title: '门店选择',
        userInfoPhone:'',
        StoreId:"",
        Store:[
          {name:"广州饰和荣威",id:"0a1a00b55bb31aa7815bb417e53d03eb"},
          {name:"广州众道大众",id:"0a1a00b55e9011a3815e9de89a2353fd"},
          {name:"广州天河WEY",id:"0a1a00b5621a170a8162238fa3e016ab"},
          {name:"广州番禺哈弗",id:"0a1a00b55ba41bd6815ba4688a1f0951"},
          {name:"广东有道哈弗",id:"0a1a00b55ba919d7815ba9dc35b30049"},
          {name:"广州友道哈弗",id:"0a1a00b55b9e184c815b9f8dbdc50fd8"}
        ]
    }
  },
  mounted() {
    //挂载完毕初始化页面数据
    this.markInit();
  },
   
  methods: {
     
    markInit() {
      //初始化页面标题
      document.title = this.title;
      //获取shopsid缓存数据
      setShopsId();
      this.$store.dispatch('log', {account:LOGCODE.SURVEY.PAGE_SURVEYINDEX});
    },
    storeBtn(pId) {
      this.StoreId = pId;
      setSessionStore("information_selected_store", {id:pId});
      this.$router.push({path: '/questionnaire'});
    }
  }
}
</script>
<style lang="less">
  @import "../../styles/common/survey.less";
</style>
